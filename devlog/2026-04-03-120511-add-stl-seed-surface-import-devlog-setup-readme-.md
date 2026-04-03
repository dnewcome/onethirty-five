# Add STL seed/surface import, devlog setup, README and PLAN updates

_2026-04-03_

## What happened

Two new STL import features landed in the phyllotaxis visualizer today. The first lets you swap out the built-in seed geometry (sphere, box, cone, tetra) with any STL file — it normalizes the mesh to unit size so the existing seed size slider still works. The second is more interesting: a new "STL" shape mode that projects the phyllotaxis spiral onto the surface of an imported mesh using a Fibonacci sphere raycasting approach — for each seed, a ray is cast from outside inward and the seed is placed at the surface intersection, oriented along the face normal. That means the golden angle distribution wraps across a heart, a face scan, or any organic form while seeds always point outward. Also wired up devlog-tools for the first time so the commit history has proper narrative entries.

## Files touched

  - .claude/commands/devsnap.md
  - .project.toml
  - PLAN.md
  - README.md
  - devlog/2026-04-03-093200-initial-commit.md
  - devlog/2026-04-03-094225-zome-dome-mode.md
  - devlog/2026-04-03-094537-zome-azimuthal-spacing.md
  - devlog/2026-04-03-094839-zome-elevation-fix.md
  - devlog/2026-04-03-100250-zome-generator-ring.md
  - devlog/2026-04-03-100801-zome-lighting-fix.md
  - devlog/2026-04-03-102822-plan-rigidity-zonelines.md
  - devlog/assets/.gitkeep
  - phyllotaxis.html
  - scripts/devlog-preview.sh
  - scripts/devpublish.sh
  - scripts/devsnap.sh
  - scripts/install-hooks.sh

## Tweet draft

Added two STL import modes to the phyllotaxis visualizer: one to replace seed geometry with a custom mesh, and one to project the golden angle spiral onto the surface of an imported shape. Load a heart or face scan and watch the Fibonacci distribution wrap around it. [link]

---

_commit: 647fb63 · screenshot: no screenshot_
