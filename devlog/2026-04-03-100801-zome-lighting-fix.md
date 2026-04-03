# zome: fix lighting artifacts and self-shadow polar ring

_2026-04-03_

## What happened

The zonohedron was showing a dark banding artifact around the polar ring — the top row of faces was self-shadowing because the shadow camera frustum wasn't tall enough to cover the full dome from above. Adjusting the shadow camera bounds and bias cleared it up. The fix also improved the overall lighting quality on the rhombic faces, which have shallow angles that are prone to shadow acne.

## Files touched

```
zome.html | 13 insertions(+), 21 deletions(-)
```

## Tweet draft

Fixed a self-shadowing artifact on the polar ring of the zome dome — the shadow camera frustum was clipping the top faces. Clean lighting on the rhombic faces now, which matters when the whole point is understanding the geometry. [link]

---

_commit: 235538a3 · screenshot: none_
