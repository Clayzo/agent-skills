# Shape construction

Marks and pictograms from primitives, so nothing is a hand-traced path unless
it has to be. Rects and ellipses are exact, cheap in both players, and free of
the arc approximation that SVG paths go through.

## Building blocks

- **Half-pill** — a circle (Ø `s`) plus a rect over the flat half
  (`s/2 × s`). Rounded on the circle's side, square on the other.
- **Cell with one or two rounded corners** — a rect with `cornerRadius` rounds
  all four; for fewer, union a circle with rects, or clip a circle by a rect.
- **D-shape** — a circle plus the quadrant rect that squares one corner.
- **Leaf, or vesica** — two quarter-discs, or a cell with opposite corners
  rounded at about 0.9 × cell.
- **Concave four-point star** — a square with four background-coloured discs
  on its corners.
- **Starburst** — `polystar`, variant `"star"`. The first outer tip sits at
  `rotationOffset − 90°` (straight up when the offset is 0) and tips repeat
  every `360 / points`. A 24-point burst at outer 206 / inner 161 is the
  campaign hero; a 10-point badge at 57 / 28.8 sits in a product card.
- **Bars of a wordmark** — one rect per bar; only curved bar ends and notches
  need paths, traced with a vertex per pixel row.
- **Quarter disc** — a circle clipped by a rect, or a `path` with two cubic
  tangents; SVG arc commands (`A`) are flattened to straight lines by the
  importer, so do not import arcs.

## Assembly rules

- Compose in the mark's own unit space (cell = 1) and scale by placing the
  group, never by baking pixel sizes into the parts.
- Overlap same-colour parts by 0.3–0.6 px so antialiased edges do not show
  the background as a seam. A rect whose edge lies inside a circle of the
  same colour needs no bleed; a rect whose edge is tangent to the circle does.
- Keep each part's origin at the point the animation is about — the junction
  of a mark, the axis a stripe grows from — so `anchor` is a constant.
- Draw grey resting copies first and coloured copies above them when a mark
  fills in; the two stacks share geometry, differ only in fill and clip.

## Colour

When matching a clip, sample the clip. Brand red `#F24E1E` played back as
`#fb5a2f` after encoding, and every other swatch shifted the same way; the
recreation should match what people see, and the swap to brand values is one
constant later.

## Lockups and clearance

- Square lockup: wordmark width 0.80–0.85 × mark width, gap 0.10–0.12 × mark
  height, and the pair centred so the mark's visual weight sits slightly
  above centre (about 55 % of the height above).
- Nothing of the hero crosses the title's line. Handles, tips, arcs and halos
  go on the side away from type, and a band that carries type is checked for
  collisions at 360 px before anything is timed.
- A stroke's width scales with the group it sits in (0.055 units in a group
  scaled ×100 is 5.5 px); set it in unit space like everything else.
- Radial `gradientFill`: `start` is the centre and `end` a point on the
  radius, both in the node's own centred coordinates.
- A halo or lining drawn as an offset arc should sit on the edge (offset ≈
  half the stroke width plus 1 px); a visible gap reads as a mistake, not air.
