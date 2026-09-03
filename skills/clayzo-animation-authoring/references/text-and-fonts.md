# Text and fonts

Type as geometry: where a `text` node actually draws, why it sometimes draws
nothing, and how to make a stand-in face match a proprietary one.

## What you cannot guess

- A `text` node's position is the top-left of its `box`. With
  `verticalAlign: "top"` the baseline sits `lineHeight` below that; with
  `"middle"` it sits half a line below the box's vertical centre.
  `align: "center"` centres the run inside `box.width`, so give the box the
  canvas width and put x at 0 to centre on the canvas.
- No font, no text. A run whose `fontAssetId` cannot be loaded, or whose
  family no loader supplies, draws nothing — no substitution, no error. The
  `text.shaping_fallback` and `text.exact_shaping_unavailable` warnings are
  normal; they mean deterministic CanvasKit metrics were used, not that the
  font is missing.
- `fontWeight` does not choose a weight. Bold is faked by emboldening at
  600 and above, and a variable font loads at its default instance. For a
  real heavy weight, instance the variable font at the weight you need
  (`fontTools varLib.instancer … wght=800`) and ship that static file as the
  asset.
- A font asset's `uri` resolves relative to the document on disk. Keep the
  file beside the document (`fonts/…`) so packaging, the terminal export and
  the preview all find it.
- Measure before trusting a size. Render one probe frame and read the run's
  pixel extents. Manrope ExtraBold sets capitals at 0.74 × size, x-height at
  0.55 ×, descenders at 0.24 ×; "Wrapped" at 101 px is 441 px wide.

## Scaling and anchoring a run

Scale about the glyphs, not the box. The anchor is in the node's local space:
`x = box.width / 2`, `y = pivot − (baseline − lineHeight)`, where `pivot` is
the canvas y you want to scale about. Anchor at the baseline for titles that
settle, so they grow upward from where they sit; anchor at the glyph centre
for numerals that pop symmetrically.

A face shorter than the one you are matching can be stretched 5–8 %
vertically (`scale.y = 1.07 × scale.x`) before it reads as distorted; wider
faces are better condensed by choosing a smaller size than by scaling x.

## Layering

A numeral behind a title but above its band: order `band → numeral → title`
and let the numeral's strokes show between the letters. It reads as depth.

## Ranges

- Poster title: cap height 0.13–0.18 × canvas width in portrait; numerals
  behind it 1.5–2 × the title's cap height.
- Letter-spacing 0 for a display sans at these sizes; tracking edits under
  0.08 em.
- Placeholder copy in product mockups: bars at 0.3–0.5 × the line height,
  one darker for the name, lighter for the rest.
