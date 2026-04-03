# zome: add generator ring diagram and alias detection

_2026-04-03_

## What happened

Added a 2D generator ring diagram that draws the azimuthal positions of all N generators as a polar plot in the panel. This makes the spacing pattern immediately legible — you can see whether generators are uniformly distributed, golden-angle-distributed, or something in between. Also added alias detection: when two generators land within a threshold of each other, the tool warns you, since near-coincident generators produce nearly-flat faces that are structurally useless and visually confusing.

## Files touched

```
zome.html | 79 insertions(+)
```

## Tweet draft

Added a generator ring diagram to the zome explorer — a small polar plot showing where each generator sits azimuthally. Plus alias detection: if two generators get too close you get a warning before the geometry degenerates. [link]

---

_commit: 40e6913c · screenshot: none_
