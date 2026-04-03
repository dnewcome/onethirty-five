# zome: fix variable elevation formula to avoid degenerate geometry

_2026-04-03_

## What happened

The variable elevation mode was producing degenerate geometry when generator elevation angles bunched up near 0° or 90°. The formula was mapping elevations linearly across all N generators but not guarding against the poles, which collapsed adjacent vertices into the same point and produced NaN normals that broke the lighting. A small clamp on the elevation range fixed it — the geometry stays well-behaved across the full slider range now.

## Files touched

```
zome.html | 3 insertions(+), 2 deletions(-)
```

## Tweet draft

Tracked down a degenerate geometry bug in the zome variable-elevation mode — collapsed vertices near the poles were producing NaN normals. One clamp fixed it. The twisted tapered forms it enables are worth it. [link]

---

_commit: 7da785bc · screenshot: none_
