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
frames; a seam that matches its neighbours is a seam nobody will see. The
printed `verdict` is `closes` when the seam is within 1.6× the larger frame
step beside it, and `steps` otherwise; `review` judges the loop point the
same way.

Worth running rather than judging by eye: a step that reads as a flicker in
motion is often invisible in a contact sheet, because you are looking at two
frames that are not adjacent on screen.

## Diagnostics to inspect
- Loop seam RGBA difference, start/end property values, adjacent seam motion energy, spring settle estimates, flashes, and alpha edges.

## Anti-patterns
- Duplicating the terminal frame, returning position without matching velocity, looping an unsettled spring, or using crossfade to hide unrelated motion.

## Parameter ranges
- Seam evaluation should include `start`, `start+1`, `end-1`, and `end`.
- `--seam` should say `closes` unless the loop cuts on purpose.

`review` reports a seam as `motion.loop_seam` and names the nodes that jump.
`self-review.md` says how to mark an intentional cut or a piece that plays once.

## Adaptation
Choose cyclical paths for continuous phenomena and explicit cuts for narrative beats. Adjust midpoint and easing from observed seam energy rather than applying a generic ping-pong.
