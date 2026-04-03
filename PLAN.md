# Phyllotaxis Lab — Extension Plan

This document outlines planned features for `phyllotaxis.html`, organized by theme.
Each section describes the idea, technical options, trade-offs, and implementation notes.

---

## 1. Mesh Export (STL)

**Goal:** Export the current grown structure as a solid 3D mesh for fabrication,
further modeling, or external rendering.

### How it works

Three.js ships `STLExporter` (`three/addons/exporters/STLExporter.js`). It accepts
a `Scene`, `Object3D`, or `Mesh`. **InstancedMesh is not directly exportable** — the
instances must first be merged into a single `BufferGeometry`.

Merge strategy per instance `i`:
```
geo_i = seed geometry.clone()
geo_i.applyMatrix4(instancedMesh.getMatrixAt(i))
merge all geo_i → BufferGeometry (using BufferGeometryUtils.mergeGeometries)
STLExporter.parse(mergedMesh, { binary: true })
```

Binary STL is ~6× smaller than ASCII and universally supported (Blender, Fusion 360,
PrusaSlicer, etc.).

### Options

| Option | Notes |
|---|---|
| Binary STL | Compact, standard. No color support. Best for fabrication. |
| ASCII STL | Human-readable, larger. Useful for debugging. |
| OBJ + MTL | Supports vertex colors. Blender imports well. More complex to generate. |
| PLY (mesh) | Supports vertex colors AND normals. Blender native. Worth considering. |

### Recommendation

Offer **binary STL** as the primary export. A secondary **ASCII PLY** (with per-vertex
color baked from instance colors) would preserve the gradient/rainbow coloring in Blender.

### Notes

- Only export visible (grown) seeds, not the full capacity of 1000
- Merged geometry for 1000 box seeds ≈ 6000 faces — very manageable
- Merge is a one-time CPU op triggered by button press, not per-frame

---

## 2. Point Cloud Export (XYZ / PLY)

**Goal:** Export seed positions (and optionally orientations) as a point cloud for use
in lighting simulation, fixture layout, procedural placement in other software.

### The Fixture Use Case

For directional light fixtures, what matters is:
- **Position** (x, y, z) — where the fixture sits in space
- **Normal / aim vector** — which direction it points (outward from center, or along
  the surface normal of the growth surface)

The seed quaternion we already compute encodes this orientation. Extracting the "aim"
axis from the quaternion gives the outward normal vector per seed.

### Formats

**XYZ ASCII** (`.xyz` or `.pts`) — simplest possible:
```
x y z
x y z
...
```
With normals:
```
x y z nx ny nz
x y z nx ny nz
```
Blender imports this via Point Cloud Visualizer add-on. Some CAD tools accept it raw.
This is the most universally understood format for point data.

**ASCII PLY** — Blender 3.x/4.x imports natively, no add-on required:
```
ply
format ascii 1.0
element vertex 300
property float x
property float y
property float z
property float nx
property float ny
property float nz
property uchar red
property uchar green
property uchar blue
end_header
x y z nx ny nz r g b
...
```
Supports position + normals + color in one file. Best for Blender workflow.

**CSV** — compatible with everything (Excel, Python, Houdini, etc.):
```
x,y,z,nx,ny,nz,r,g,b
```

### Recommendation

Implement all three with radio buttons. The normal vector per seed is the Y-axis of
the seed's local transform (the "up" direction before rotation), transformed by the
instance quaternion. Extract it as:
```js
const normal = new THREE.Vector3(0, 1, 0).applyQuaternion(seed.quat);
```

---

## 3. Custom STL Seed Geometry (Import)

**Goal:** Replace the built-in primitives (sphere, box, cone, tetra) with a user-supplied
STL file. This allows custom seed shapes — a diamond, a leaf, an architectural element,
a light fixture body, etc.

### How it works

```
<input type="file" accept=".stl">
  → FileReader.readAsArrayBuffer()
  → STLLoader.parse(buffer)  →  BufferGeometry
  → center + normalize geometry to unit size
  → use as InstancedMesh geometry (replaces current)
```

`STLLoader` returns a `BufferGeometry` (non-indexed, with normals computed). Both
binary and ASCII STL are supported automatically.

### Normalization

Imported geometry should be centered at the origin and scaled to a unit bounding box,
so the existing `seedSize` slider still controls apparent scale:

```js
geo.computeBoundingBox();
const center = new THREE.Vector3();
geo.boundingBox.getCenter(center);
geo.translate(-center.x, -center.y, -center.z);
const size = geo.boundingBox.getSize(new THREE.Vector3()).length();
geo.scale(1/size, 1/size, 1/size);
```

### Notes

- The imported geometry replaces the current geometry; all existing controls still work
- Orientation: imported seeds will be rotated by the same outward-facing quaternion
  as built-in shapes — may need a user-adjustable "seed orientation offset" (Euler XYZ)
  to align imported geometry's "front" to the outward axis
- Memory: large STL imports (> 100k faces) will be slow for 1000 instances; warn user

---

## 4. Surface-Mapped Phyllotaxis

**Goal:** Import an arbitrary mesh surface (e.g., a heart, a sphere, a face scan) and
map the phyllotaxis spiral onto it — so seeds grow across the surface of that shape
rather than a mathematical cone/cylinder.

This is the most visually striking extension: the golden angle arrangement adapts to any
surface topology.

### Approach A — Closest-Point Projection (simplest, recommended first)

1. Compute phyllotaxis positions on a reference shape (sphere or flat disk)
2. Scale reference shape to match target mesh bounding box
3. For each seed position, cast a ray from the center outward; find intersection with
   target mesh surface
4. Place seed at intersection point with surface normal as orientation

```js
const raycaster = new THREE.Raycaster();
raycaster.set(origin, direction.normalize());
const hits = raycaster.intersectObject(targetMesh);
if (hits.length) {
  position = hits[0].point;
  normal = hits[0].face.normal; // transformed to world space
}
```

**Trade-offs:** Simple and fast. Works well for convex shapes. Fails for concave
surfaces where rays miss or double-hit (e.g., inside of a bowl).

### Approach B — Surface Sampling with MeshSurfaceSampler

Three.js `MeshSurfaceSampler` (`three/addons/math/MeshSurfaceSampler.js`) samples
random points on a mesh surface proportional to face area, returning position and normal.

This does **not** preserve the golden angle ordering — it's random. However, you could:
- Sample N points randomly
- Re-order them by a surface parameterization (e.g., distance from a pole)
- Apply the golden angle to that ordering

This is harder but gives true surface-respecting distribution.

### Approach C — UV-Space Phyllotaxis (most mathematically correct)

If the target mesh has UV coordinates:
- Compute phyllotaxis positions in 2D UV space (0..1, 0..1)
- Use `MeshSurfaceSampler` or inverse UV lookup to convert UV → 3D position + normal
- Place seeds there

Works perfectly for meshes with clean UV maps (most CAD exports). Requires UV data in
the STL — which standard STL does not have. Would need OBJ format instead.

### Recommendation

Implement **Approach A** first. It works for the heart, sphere, organic shapes. Import
via STL + STLLoader; use raycasting from a bounding sphere outward. Add a "reference
shape" selector (sphere projection vs cone projection) that determines the origin of rays.

---

## 5. Developable Surface Panels from Spiral Arms

**Goal:** Generate flat (planar) panels bounded by adjacent spiral arms — pieces that
can be cut from flat sheet material (cardboard, plywood, metal sheet) and assembled into
the 3D phyllotaxis form.

This is directly applicable to physical fabrication: laser cutting, CNC routing.

### The Mathematics

A **developable surface** has zero Gaussian curvature — it can be unrolled flat without
stretching. Cones and cylinders are the two classic developable surfaces. Both appear
in the phyllotaxis system:

**Cone (pinecone mode):**  
A cone with half-angle α unrolls into a flat sector. The sector has:
- Radius = slant height L of the cone
- Central angle = 2π · sin(α)

Every point (h, θ) on the cone maps to polar coordinates in the flat sector:
```
ρ = h · L / H          (radial distance, where H = total height)
φ = θ · sin(α)         (angular position)
```

The Fibonacci spiral arms on the cone become **straight radial lines** on the unrolled
sector. The "cells" between adjacent spiral arm pairs become flat trapezoidal or
rhombus-shaped panels — these are the pinecone scales, exactly, but flat.

**Cylinder (pineapple mode):**  
A cylinder unrolls into a flat rectangle (width = 2πr, height = H). The spiral arms
become diagonal straight lines. Adjacent arm intersections form parallelogram cells.

### What to Generate

For each pair of adjacent Fibonacci spiral arms (F₁ and F₂), extract the boundary
cells and output them as:
- **DXF** (CAD standard for laser cutting)
- **SVG** (universal flat-panel format)
- **OBJ** (flat quads, importable anywhere)

Each panel/cell is bounded by four seed positions (quadrilateral). For the cone, the
four corners of each cell are seed positions `[i, i+F₁, i+F₂, i+F₁+F₂]`. The cell
is nearly planar (developable approximation) and can be output as a flat quad.

### Panel Tabs and Assembly

For physical fabrication, add:
- Small rectangular tabs on alternating edges for gluing
- Score lines along fold edges
- Numbering / registration marks per panel

### Limitations

The cells are only **approximately** flat — true planarity requires computing the
developable approximation of each cell individually (fitting a plane to the four corner
points). For a true cone or cylinder, they are exactly flat. For the cylinder mode
with envelope (pineapple), the surface is not developable; panels would need to be
kerf-bent or vacuum-formed.

### Recommendation

Start with **cone mode only** (true developable). Compute the 2D unrolled coordinates
for all seed positions, draw the quad cells in SVG, and export as a flat cut pattern.
This alone is a complete, self-contained fabrication pipeline.

---

## Implementation Priority

| Feature | Complexity | Value | Priority |
|---|---|---|---|
| STL export | Low | High | 1 |
| XYZ / PLY point cloud export | Low | High | 2 |
| Custom STL seed import | Medium | High | 3 |
| Surface mapping (projection) | Medium | High | 4 |
| Developable cone panels (SVG) | Medium | Very High | 5 |
| UV-space surface mapping | High | Medium | 6 |
| True MeshSurfaceSampler mapping | High | Medium | 7 |

Features 1–3 are all self-contained additions to `phyllotaxis.html`.  
Feature 4 (surface mapping) could be a separate page `surface.html`.  
Feature 5 (developable panels) is a strong candidate for a dedicated `fabricate.html`.

---

## Format Reference

| Format | Supports | Blender | CAD | Notes |
|---|---|---|---|---|
| Binary STL | mesh | native | native | No color. Best for fabrication. |
| ASCII PLY | mesh + normals + color | native | partial | Best Blender mesh export. |
| XYZ ASCII | points (+ normals) | add-on | most | Simplest. Universal. |
| CSV | points + any data | via script | Excel | Best for data analysis / Houdini. |
| PLY (points) | points + normals + color | native 4.x | partial | Best Blender point export. |
| DXF | 2D curves | via add-on | native | Best for laser cutting. |
| SVG | 2D curves | via Inkscape | partial | Human-readable flat patterns. |
| OBJ | mesh + normals | native | wide | Good fallback mesh format. |
