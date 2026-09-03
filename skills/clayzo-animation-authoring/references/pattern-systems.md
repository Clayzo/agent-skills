# Pattern systems

Backdrops built from many copies of a few shapes: tiled marks, diagonal
stripes, pixel chains, dot rings, disc grids. The pieces are trivial; the
coordinate system is the work, and it is where hand-placed layouts drift.

## Decision principles

- Index cells, not pixels. Place every unit at `origin + step × (i, j)` and
  derive screen positions from integer cell coordinates, so the whole system
  is tuned by three numbers (origin, step, unit size) instead of hundreds.
- Put orientation in the transform, not the geometry. Build the unit once in
  its own local space with `anchor` on the point that stays put, then place
  copies with `position`, `rotation.z`, and `scale.x = -1` for mirrors. A 45°
  lattice is an axis-aligned lattice plus `rotation.z: 45`.
- Choose the pivot the motion is about, then make `anchor` and `position`
  both name it. A stripe that "zooms in from the screen's axis" scales about
  the point where it crosses that axis; a mark that "floods from its corner"
  opens about that corner.
- Decide which layer owns opacity. Units that overlap and fade together must
  live in one group with `isolation: true`, or every overlap doubles up while
  the group is translucent.
- Cull generously, not exactly. A unit just outside the frame at rest slides
  into view while its group is small or turning; keep anything within about
  120 px of the frame edge.

## Recipes, with the numbers that worked

**Diagonal lattice of a mark.** Cell size `s`. Cell centres at
`(x0 + h·a, y0 + h·b)` with `h = s/√2` and `a + b` even — a square lattice
rotated 45°. Each mark is a group anchored on its junction corner and placed
at a lattice vertex. Three orientations reproduce a brick tiling:
`{ rotation: 45 }`, `{ rotation: 45, scaleX: -1 }`, `{ rotation: 225 }`;
alternate them along the diagonal band and mirror across the band's
perpendicular. Fit `x0, y0, h` by least squares on a few clean circles rather
than by eye — a 0.8 px error in `h` is a 6 px error eight cells out.

**Pixel chains.** 88 px squares on a 44 px lattice at cells `(r + d, r)` for
`r = 0…16` and diagonals `d ∈ {−18, −12, −6, 0, 6}`: half-overlapping squares
read as a staircase ribbon; six cells of `d` is one ribbon spacing (264 px).
Keep the chains continuous and let the foreground hero occlude them — the
"glyphs" a viewer sees are the occlusion, not gaps.

**Disc grid behind the chains.** A 3 × 5 grid at 245 × 249 px pitch, radius
62, drawn under the chains so only the notches show colour. Edge discs are
centred 14 px outside the frame so they read as half-discs on the same column
pitch.

**Ring of punctuation.** 14 dots of Ø50 on the perimeter of a 5 × 5 grid at
~104 px pitch, minus the two mid-side points the title band would cover. They
arrive in a measured, irregular order (see `motion-recipes.md`), never a
clockwise sweep.

**Stripes of a wordmark.** One rect per bar per letter, eight bars at equal
pitch; only the curved bar ends and the notch need paths. The two halves of a
split-field piece are not a rigid copy of each other — measure each.

## Seams

Quote bleed in the pixels of the smallest render you will inspect: 0.6 px at
1080 is 0.2 px on a 360 px sheet, where both antialiased edges fall in one
pixel and every join shows a hairline. Use 0.3–0.6 px for the deliverable
and expect hairlines on the small sheet; if the deliverable itself is small,
bleed 1–2 px. Bleed interior edges only — a symmetric bleed on edge cells
overhangs the canvas.

Two same-colour shapes that abut show the background as a hairline where
both antialiased edges meet, and a circle tangent to a flat edge shows a
sliver for a stretch either side of the tangent point. Overlap by a fraction
of a pixel — rects grow 0.6 px, circles 0.3 px — rather than trusting
tangency. Different colours abutting show the same faint light line; the same
bleed hides it, and a 0.5 px colour overlap is invisible.

## Anti-patterns

- Hand-placing tiles by screen coordinate, then chasing a drift of a few
  pixels per column.
- One rotated group for the whole pattern when units animate independently;
  you lose the per-unit pivots.
- Per-node opacity on overlapping units.
- A pattern that ends exactly at the frame edge: fine at rest, wrong the
  moment anything zooms or slides.

## Parameter ranges

- Tile pitch 1.0–1.1 × unit for touching tilings; 2.0–2.5 × unit for
  punctuation grids (dots, discs).
- Ribbon spacing 3 × unit for pixel chains.
- Bleed 0.3–0.6 px.
- A backdrop system should use at most two unit sizes; the reference tilings
  here use one.

## Visibility, budget, drift

- A backdrop that cannot be seen in a 360 px sample sheet does not exist for
  the viewer: keep units at least 8 px at that size and at least 12 %
  luminance contrast against the field. Vignettes that fade a dot field to
  nothing at the edges are fine; a whole field at 7 % contrast is not.
- Several hundred small units, each with pop keyframes, take every slot in
  the review sampler, and the hero, title and punctuation go unreviewed.
  Author beat markers anyway, and check the beats with your own
  `render-frame --samples` sheets.
- A slow drift of the whole lattice is motion from its first tick. If the
  piece opens on a still, start the drift after it.

## Cost

A unit's node `opacity` is a layer per unit per frame; a field of a few
hundred fading dots turns a 20 ms frame into a 2 s one. Fade solid units
through `style.fill.opacity`, or draw the whole field as a handful of `path`
nodes grouped by onset. Units under 8 px fade in rather than pop — a pop at
that size is invisible and costs the same.

## A hero inside a tiling

When the hero sits on a tiled backdrop, half tiles peeking around its
silhouette are the first thing wrong with the render. Three moves fix it:
a keyline clearing — the hero's own silhouette offset outward by 30–40 px
in the field colour, drawn under the hero; a lattice origin chosen so every
tile is either fully inside the clearing or fully clear of it; and culling
the one tile an overshoot would clip. The tiled mark as a *negative* of the
hero (the clearing is cloud-shaped, the cloud assembles inside it) is a
stronger idea than a plain lattice, at no extra cost.
