# Text and fonts

A wordmark of up to six letters in a geometric sans is a fifteen-minute job
from rings and pills — two rings for the o's, the same stroke bent into an L
or a p — and needs no font at all. When no brand face is supplied, prefer
that: it makes the name part of the system, and it packages.

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
- The package ships no font file. Put a permissively licensed face (an OFL
  family such as Manrope or Inter) in `fonts/` beside the document and point a
  `font` asset at it. System faces render locally but cannot be embedded or
  packaged, and fail on any other machine.
- `fontAssetId` selects the file. The run's `fontWeight` still emboldens at
  600 and above, so set it to 400–500 on a run whose asset is already a bold
  face; a `.ttc` collection loads its first face and there is no face index.
  Leave `letterSpacing` at 0 (centring ignores tracking) and `verticalAlign`
  at `"top"`.
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

## Measuring a face

One probe document: the run at the size you intend on a plain field, a `rect`
hairline at the intended baseline, rendered with `render-frame --tick 0`. Read
the pixel extents of the glyphs against the hairline; that gives cap height,
x-height, descender and width for the real face in three minutes, and the
numbers go in a comment at the top of the script.

## Platform safe zones

- Instagram Story and Reels, 1080 × 1920: the app covers roughly the top
  250 px and the bottom 220 px. Keep everything that matters between y 260
  and y 1700, and titles at least 6 % of the width from the sides.
- TikTok, 1080 × 1920: the same vertical band plus a right rail of about
  120 px for the action column.
- Square feed posts have no chrome; keep 5 % margins.
