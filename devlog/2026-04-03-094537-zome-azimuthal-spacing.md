# zome: replace golden toggle with azimuthal spacing controls

_2026-04-03_

## What happened

Replaced the binary "golden angle spacing" toggle with a continuous azimuthal spacing slider. The original toggle just switched between uniform 360°/N and exactly k×137.5° — but the interesting territory is the space between those two values. With a continuous control you can explore how the face shapes deform as spacing becomes more irregular, and find intermediate configurations that are visually compelling but don't correspond to either pure system. This is the unexplored hybrid zone between phyllotaxis and zonohedra.

## Files touched

```
zome.html | 79 insertions(+), 17 deletions(-)
```

## Tweet draft

Replaced the golden-angle toggle on the zome explorer with a continuous spacing slider. The interesting zone isn't either endpoint — it's the deformed, irregular configurations between uniform and golden that don't have a name yet. [link]

---

_commit: 97194aad · screenshot: none_
