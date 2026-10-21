# Self-review

## Decision principles
- Review the exact current revision with `clayzo review`, and look at the contact sheet and frames it writes, not only the counts.
- Treat diagnostics as evidence: fix errors, fix or explain every warning, and mark intent in the document where a check offers a way.
- Re-review after every edit; never carry approval across revisions.

## Running it

```bash
clayzo review hero.json review/      # review.json, contact-sheet.png, frames/
```

stdout ends with a summary:

```json
"diagnostics": { "errors": 0, "warnings": 2, "codes": {
  "motion.pop_in": { "severity": "warning", "count": 1, "nodeIds": ["badge"] },
  "text.low_contrast": { "severity": "warning", "count": 1, "nodeIds": ["caption"] } } }
```

- `codes` lists each warning and error code with up to 12 node ids. Every
  diagnostic, info included, is in `review.json → analysis.diagnostics` with
  its `message` (what happened and what to change), `ticks`, `nodeIds`,
  `region` (composition px) and `evidence`. Renderer messages per frame are in
  `frames[].rendererDiagnostics`, graded by `level` where analysis
  diagnostics use `severity`; the per-frame timings and cache notes are
  counted in `rendererCounts`.
- stdout also prints `sampledTicks`, with `sampledFor` (why each was taken)
  and `omittedMarkers` (markers with no frame, when any), and `measured`,
  which review.json has too: `firstChangeTick` (the
  first frame unlike frame 0) and `stillSeconds` before it, `lastChangeTick`
  (where the final picture arrives) and `endHoldSeconds` after it, and `loop`
  (`declared`, and `seam` against `frameStep` with a `verdict` when it was
  measured). The times are left out when the checks cannot place every node.
  Check them against the plan even when nothing warns.
- Motion, layout and text checks read every displayed frame. Pixel checks read
  the contact sheet's frames, the loop point, and frames where a large layer
  changes at once. The sheet keeps the first and last frames, a frame per
  marker (and the frame before a cut, while there is room), and three where
  the picture changes most: 10 frames, up to 12 to fit the markers, or
  `--frames <n>` up to 24, at 512 px unless `--max-dimension` or
  `--full-resolution`. Each frame is labelled with its tick, time and why it
  was taken. Put a marker on every beat you need to see, and span a run of
  cuts with one marker.

## Codes

| Code | Means | Fix |
|---|---|---|
| `pixels.blank` (error) | A sampled frame draws nothing. | Check `rootNodeIds`, `inTick`/`outTick`, opacity, fonts. |
| `renderer.unresolved` | The renderer warned or failed on a sample. | Read `frames[].rendererDiagnostics`. |
| `pixels.edge_clipping` | Artwork touches the frame edge on a frame it does not fill. | Inset it, or draw a background. |
| `layout.edge_gap` | A background meant to fill the canvas stops a pixel or two short of an edge: a hairline border, black in video. | Size the field to cover the canvas; shift the content, not the background. |
| `pixels.alpha_rgb_leak` | Fully transparent pixels carry colour. | Return premultiplied colour from custom shaders. |
| `motion.pop_in`, `motion.pop_out` | A part appears or vanishes at full opacity in one frame. | Fade or scale it, or mark the frame as a cut. |
| `motion.flicker` | Opacity, scale or position swings and back on most frames. | Remove the overlapping keyframes; change it once. |
| `motion.shake` | Small reversals no keyframe of the node or its parents makes (two animations fighting over one property), or position, rotation or scale twitching back and forth within a few pixels, degrees or percent, even briefly. | One animation per property; hold still, or give the motion a clear swing. |
| `motion.large_flash` | Most of the picture changes at once with its brightness swinging, more than once within a second. | Ease the changes, or mark each as a cut. |
| `motion.flash_hazard` | A third of the frame each way brightens and darkens by 10 % or more over three times a second: a seizure risk (WCAG 2.3.1). No marker excuses it. | Three flashes a second or fewer, less contrast, or a smaller area. Fine noise averages out; coarse noise does not. |
| `motion.glitch` | Something shows for one frame that the motion around it does not explain: a part shown for 0.1 s or less, a one-frame jump in opacity, place, size or look, or a thin sliver a fast clip leaves. | Find the stray keyframe or the in or out tick a frame off; stop a clip's edge short of the part, or ease its last steps. |
| `motion.loop_seam` | The last frame does not lead into the first. | Close the loop (`loop-design.md`), or mark intent below. |
| `layout.pass_through` | Like-sized parts pass through each other; one comes to rest on another it was clear of; text's letters rest on other text (a subtitle on a headline) or touch a shape's edge; a small part slides straight across a larger still one; a few of a scatter of alike parts land on content while the rest sit clear. Crossings in a burst of five or more alike parts in flight pass. | Move it clear, route it around, take it behind, or fade it; a part 5 % or more larger than at rest as it passes reads as lifted over. |
| `layout.off_canvas` | Text at rest is cut off by the edge, or a group holding text rests 20 % or more off the canvas. | Move it inside. |
| `text.low_contrast` | Text is under 4.5:1 against what is behind it (3:1 from 24 px, or 18.66 px bold). | Change its colour, the backdrop, or its size. |
| `text.family_unavailable` | A family the text names isn't installed or an OFL family on Google Fonts; the message names the family it is drawn in instead. | Name a family this machine has or Google Fonts carries, or ship the face as a `font` asset. |
| `craft.missing_copy`, `craft.orb_only_subject`, `craft.missing_lock_clock` | A document tagged `product-ui`, `lock-screen` or `ios-lock-notification-stack` has no text, only ellipses, or no readable clock. | Set the copy and clock as real text. |
| `performance.*` | Web playback cost: see below. | |

Info only, never in `codes`: `pixels.low_contrast_range`,
`motion.frozen_interval`, `geometry.not_checked` (nodes the checks cannot
measure, such as nodes under a perspective camera or rotated in 3D; the
message says why), `performance.near_budget`, `performance.unavailable`,
`performance.unpackaged_fonts`, and the quiet forms of the codes above: a
single hard cut, noise that changes most pixels but not the brightness, a
piece that comes to rest and cuts back to the start, display type bleeding
up to 10 % off the edge, shapes off the canvas, and pops, flicker and
glitches in a piece built from hold cuts.

## Marking intent
- A hard cut on purpose: a marker whose `payload` has `cut: true`, at the cut
  (zero length, or spanning it). Pops, glitches and flashes inside it pass
  (a flash hazard never does), and the contact sheet shows both sides. At
  tick 0 it marks the loop point as a cut. `review.json → analysis.excused`
  lists what each marker let pass, when one did.
- `design: { tokens: { looping } }` on the document: `true` checks the loop
  point strictly, `false`
  skips it for a piece that plays once. Unset, a piece still moving into the
  loop point warns, and one that comes to rest first is a cut.
- Anything else that stays: name the code, the nodes, and why it is right for
  this design and revision.

## Web playback cost

`review.json → performance` says how the document plays in
`@clayzo/webgl-player`, and stdout's `performance` gives one line per device:
`holds 60 fps`, `near its budget: …`, or `drops frames: ~72% of 60 fps at 75%
resolution`.

- Rows: M-series Mac, Mid Windows laptop, Old Intel laptop / Chromebook, Mid
  Android phone, iPhone. Each has a `verdict`, frame times, `hotspots` (nodes
  or effect kinds costing 10 % or more of its GPU time) and `slowTicks`.
- `performance.frame_budget` warns only for the Old Intel laptop / Chromebook
  and Mid Android phone rows; for the Mac, Windows laptop and iPhone it is
  info, a scaled estimate.
- Laptop rows assume the page shows the animation 720 CSS px wide; pass
  `--display-width <px>` with the real width. Phones use their screen width.
  An interactive document is costed at rest.
- Review the `.clayzo` you ship (every command takes one): its fonts are what
  the page gets. In a loose document, text is costed in the faces `clayzo
  package` would embed and noted as `performance.unpackaged_fonts` (info:
  run `clayzo package`).
- `performance.download_size`: the document (gzipped), its assets and the font
  files its families resolve to come to over 3 MB; the message names the
  largest. `performance.oversized_image`: a picture over 4× the document's
  pixels and 512 KB; resize it.

When `performance.frame_budget` fires, its message names the costliest node or
effect kind and where the time goes.
- GPU time is pixels shaded. Glass, refraction, custom shaders, dithering,
  duotone, mesh gradients and large blurs or shadows cost the most per pixel
  they cover, and many of one kind add up: make them cover less (the card,
  not the canvas), use fewer, or drop the named one. Every node with opacity
  below 1, isolated group, blend mode and matte composites a layer.
- Fewer pixels: show it smaller on the page, or cap the player's
  `maxPixelRatio` below 2. Keep `resolution` at `"display"` so the player
  can lower its resolution on a slow GPU.
- When the CPU runs late, lower resolution does not help: use fewer nodes.
- A 30 fps document has twice the frame time of a 60 fps one.

`performance.not_drawn` means the WebGL player would skip content; for text,
the message names the nodes and the family nothing supplies or the font asset
that would not load. Package a face you may ship, or fix the asset's `uri`.
Change the Lottie nodes or other constructs it names, or ship
`@clayzo/canvaskit-player`.
