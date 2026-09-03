# Reveals and masks

Colour or content that arrives by being uncovered rather than moved. The
engine has two tools for it — a group's `clipPathId` and a `matte` node — and
one that is usually better than either: layering something opaque on top.

## The clip contract

- `clipPathId` names a `rect`, `ellipse`, `arc`, `polystar` or `path` node.
  Any other kind warns `clip.unsupported_node` and the group draws unclipped.
- The clip node is **not** in the group's `children` and **not** in
  `rootNodeIds`. It is reachable only through the clip; listing it anywhere
  draws it as a solid shape.
- The clip node's transform applies in the group's coordinate space, so
  position it where the reveal starts, in the same units as the children.
- Animate the clip's `size`. A zero-size clip draws nothing, so the revealed
  shape can exist from tick 0 and appear only as the clip opens.
- One level of clipping is fine for the WebGL player; nested clips and mattes
  need CanvasKit, and `checkCoverage` will say so. Prefer a clip over a matte
  whenever an outline is all you need.

## Recipes

**Corner flood.** A grey copy of the shape underneath; the coloured copy
inside a group whose clip is a circle centred on the corner the flood starts
from. Diameter track `0 → 2 × reach`, where reach is the farthest point of the
shape from that corner plus about 2 %, so the flood lands exactly as the front
clears the far edge. For a unit cell with a semicircular end, reach is
`(√2/2 + 1/2) × cell`. Reverse the track to drain. Fronts jump: a third of the
travel happens in the first frame or two, so the ease is a steep ease-out
`{ 0.2, 0.45, 0.45, 1 }`; drains accelerate, `{ 0.5, 0, 0.7, 0.7 }`.

**Sequential hand-off.** Each flood starts on the tick the previous lands
(56, 40, 44, 44, 52 ticks at 240 tps in the reference mark), and the drain
runs in reverse with a 36-tick stagger and a ~30-tick collapse. A landing
that starts the next element reads as one motion; an even stagger reads as a
machine.

**Band opening under a title.** A background-coloured rect widening about its
centre while the title fades in above it. No clip is involved — the band
reveals what the hero behind it hides. Its half-width, measured at 1/60 s from
a 0.65 s start: 25, 53, 95, 127, 151, 169, 183, 191 px, then a slow creep to
full over another 0.25 s. Trace it as linear segments; no single bezier has
that tail.

**Wipe from an edge.** A clip rect anchored on the leading edge with
`size.width` animated. Pair with a settle on the content (scale 1.12 → 1.0)
so it lands rather than stops.

**Occlusion instead of a mask.** If a foreground element covers the seam — a
starburst over stripe ends, a band over the middle of numerals — do not mask.
Keep the background continuous and let the layering do the work; it is
cheaper and it survives the WebGL player.

## Anti-patterns

- Putting the clip node in `children`: it draws as a solid.
- Clipping the grey base as well as the coloured layer.
- A reach equal to the geometry exactly: the last antialiased pixels finish a
  frame late and the landing looks soft.
- A `matte` where a `clipPathId` would do. Mattes cost a layer per node and
  lock the document to CanvasKit.
- Revealing by fading. A flood, wipe or band has a front; a fade has none, and
  the reference pieces never fade a fill in.
