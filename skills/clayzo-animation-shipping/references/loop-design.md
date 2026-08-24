# Loop design

## Decision principles
- Match first and last state exactly while preserving velocity continuity when the motion is continuous.
- Hide unavoidable discontinuity behind an intentional cut, occlusion, or hold.
- Review seam pairs, not only the first and last isolated frames.

## Measuring the seam

```bash
clayzo render-frame loop.json seam.png --seam --max-dimension 400
```

Renders the last displayed frame and the first, writes them side by side, and
reports how far apart they are — against an ordinary frame step, which is the
comparison that means something. A busy animation moves a lot between any two
frames; a seam that matches its neighbours is a seam nobody will see.

```
closes   seam=0.1806   ordinary step=0.1711
steps    seam=15.8393  ordinary step=0.0165
```

Worth running rather than judging by eye: a step that reads as a flicker in
motion is often invisible in a contact sheet, because you are looking at two
frames that are not adjacent on screen.

## Diagnostics to inspect
- Loop seam RGBA difference, start/end property values, adjacent seam motion energy, spring settle estimates, flashes, and alpha edges.

## Anti-patterns
- Duplicating the terminal frame, returning position without matching velocity, looping an unsettled spring, or using crossfade to hide unrelated motion.

## Parameter ranges
- Seam evaluation should include `start`, `start+1`, `end-1`, and `end`.
- Keep seam difference below `0.02` normalized RGBA unless the loop contains an intentional cut.

## Adaptation
Choose cyclical paths for continuous phenomena and explicit cuts for narrative beats. Adjust midpoint and easing from observed seam energy rather than applying a generic ping-pong.
