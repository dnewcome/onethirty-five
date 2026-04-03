# onethirty-five

Visualization experiments based on biological generative systems — specifically the
137.5° golden angle rotation that governs spiral phyllotaxis in plants (pinecones,
pineapples, sunflowers, ferns) and its relationship to polar zonohedron geometry.

All tools are single self-contained HTML files. No build step, no server required —
just open in a browser.

---

## Tools

### `phyllotaxis.html` — Phyllotaxis Visualizer

Interactive 3D growth simulator for spiral phyllotaxis patterns using Three.js/WebGL.

The core idea: place seed element *i* at angle `i × φ` and radius `√i × scale`, where
φ is the golden angle (~137.5°). Varying the angle, growth mode, and geometry produces
the full range of botanical forms.

**Features**
- Five growth modes: flat disk (sunflower), tapered cone (pinecone), cylinder (pineapple),
  fern frond, and **STL surface** — projects the phyllotaxis spiral onto an imported mesh
- Four built-in seed primitives: sphere, cone, box, tetrahedron
- **Custom STL seed geometry** — import any STL to use as the seed element; normalized
  to unit size so existing size slider still controls scale
- Animated growth — seeds appear one by one with spring easing
- Live controls: golden angle (90–180°), seed count, radius, height step, taper, size
- Three color modes: single, gradient (A→B by index), rainbow
- Fibonacci spiral arm overlay — dynamically computes and draws the two expressed
  Fibonacci spiral families for the current N and angle
- Presets: Sunflower / Pinecone / Pineapple / Fern
- OrbitControls (drag to rotate, scroll to zoom)

**STL surface mode**

Load any closed STL mesh as the growth surface. Seeds are distributed using a Fibonacci
sphere (equal-area projection), and for each seed a ray is cast outward from the shape's
center to find the surface intersection. Seeds are placed at those points and oriented
along the surface normal — so they always point outward from the imported shape regardless
of its geometry. Works well for convex and mildly concave shapes (hearts, faces, vases,
architectural forms). The golden angle slider still controls the phyllotaxis distribution
pattern on the surface.

**The golden angle**

137.5077...° = 360° × (1 − 1/φ) where φ = (1+√5)/2

This irrational angle ensures no two seeds ever share a radial spoke, producing maximum
packing efficiency. At 137.5° the visible spiral count follows Fibonacci pairs (8/13,
13/21, 21/34...). Drag the angle away from 137.5° and watch the spiral arms collapse
into coarser spoke patterns.

---

### `zome.html` — Polar Zonohedron / Zome Explorer

Interactive generator and exporter for polar zonohedra — the geometric basis of Zome
dome architecture as explored by Bob Bell, George Hart, and Burning Man structures.

A polar zonohedron with N generators produces N(N−1) rhombic faces organized into N−1
zone bands. Known special cases: N=3 → rhombohedron, N=4 → rhombic dodecahedron,
N=6 → rhombic triacontahedron.

**Features**
- Controls: N generators (3–24), elevation angle φ, scale
- Fibonacci quick-picks: N = 3, 5, 8, 12, 13, 21
- Display modes: solid / wireframe / both
- Color modes: by zone band (reveals helical structure), by depth, flat
- Live structure stats: face count, vertex count, band count
- Export: binary STL, OBJ, XYZ face centroids with normals

**Experiments**
- **Golden angle spacing** — replaces uniform 360°/N azimuth with k×137.5° between
  generators. The faces are no longer rhombuses; the shape remains convex but becomes
  irregular. This is the unexplored hybrid between the two systems.
- **Variable elevation** — each generator at a different polar angle (φₖ = k×90°/N),
  creating twisted tapered forms
- **Variable length** — generator magnitudes vary by golden ratio powers

**Face centroid export**

The XYZ export writes each face's centroid position and outward normal — directly
useful for spatial fixture placement simulations where the face normal defines the
aim direction of a directional element (light, speaker, antenna, etc.).

---

## Plans and Ideas

See [`PLAN.md`](PLAN.md) for the full roadmap, including:

- STL and PLY mesh export from the phyllotaxis visualizer
- Point cloud export (XYZ/PLY with normals) for fixture placement workflows
- Developable surface panels — unroll the phyllotaxis cone into flat cut patterns
  for laser cutting / fabrication (the Fibonacci spiral arms become straight lines
  on the unrolled sector)
- Zome zone line visualization and structural rigidity analysis

---

## Background

- [Patterns in nature — Wikipedia](https://en.wikipedia.org/wiki/Patterns_in_nature)
- [Phyllotaxis — Wikipedia](https://en.wikipedia.org/wiki/Phyllotaxis)
- [Zome architecture — Wikipedia](https://en.wikipedia.org/wiki/Zome_(architecture))
- [George Hart — Polar Zonohedra](https://www.georgehart.com/virtual-polyhedra/zonohedra-info.html)
- L-systems, Fibonacci spirals, zonohedra, developable surfaces
