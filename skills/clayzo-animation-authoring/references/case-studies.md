# Case studies

Four references recreated to within a frame and a couple of pixels, reduced
to what an author needs to build the same kind of piece from a brief. Every
number was measured, not chosen.

## Brand mark reveal (portrait, 3.2 s, five-piece mark on a tiled backdrop)

- **System.** The mark is five cells (two half-pills, a half-pill, a circle,
  a D) on a 2 × 3 grid, cell 80 px. The backdrop is the same mark, cell
  92.5 px, on a 45° lattice (`pattern-systems.md`) in two bands with three
  orientations; the five colours are the mark's own.
- **Motion.** Every cell floods from the mark's centre junction (the D from
  its own top corner) in sequence — red, salmon, purple, blue, green — each
  starting as the previous lands, then drains in reverse after a 1.2 s hold.
  The backdrop marks flood in sync. First and last frames are identical grey.
- **Timing.** In: 12→72, 72→112, 112→156, 160→200, 208→256 ticks. Out:
  552→584, 588→620, 624→656, 660→692, 696→728. Flood ease
  `{0.2, 0.45, 0.45, 1}`; drain `{0.5, 0, 0.7, 0.7}`.
- **Quirk worth keeping.** The green piece is 5.5 % larger than the others,
  anchored at its top-right corner.

## Campaign poster (portrait, 6 s, pixel chains, starburst, title, numerals, dots)

- **System.** Five diagonal chains of 88 px squares on a 44 px lattice; a
  3 × 5 disc grid behind them; a 24-point starburst; a background-coloured
  band; a title in an 800-weight geometric sans at 101 px; numerals at 209 px
  behind the title; 14 dots on a 5 × 5 perimeter.
- **Motion.** Chains zoom in from where each crosses the frame's anti-diagonal,
  centre chain first (0.085 s), neighbours at 0.125 s, outer chains from
  0.10 s at half speed, all fading in. Discs pop bottom-right → top-left at
  50 ms per column, 100 ms per row. Star pops to 1.18 × at 0.68 s, holds,
  snaps to size at 0.9 s. Band opens from 0.65 s while the title fades and
  settles from 1.12 ×. Numerals rise from 0.97 s with one bounce peaking at
  1.6 s. Dots pop 1.30 → 1.90 s in a measured irregular order.
- **Layering.** discs → chains → star → band → numerals → title → dots. The
  numerals' strokes show between the title's letters.

## Product card feed (portrait, 2 s, two stacked social cards)

- **System.** White cards with 12 px corners and a soft drop shadow on a blue
  page with a faint 50 px grid; header, a 4 × 2 hero band of rounded cells,
  footer of placeholder bars. The second card is the first with an orange
  palette, 461 px lower, two ticks behind.
- **Motion.** Hero cells scale and fade in about their own centres on an
  exponential approach (τ ≈ 0.3 × duration), 16–40 frames each, staggered by
  0–3 frames; the badge starburst grows from nothing while spinning 155°
  clockwise and settles at frame 90 with a point due right. Avatar slides up
  5.5 px; name bars slide in 14 px; menu dots appear right to left 12 frames
  apart; reaction circles pop 6 frames apart; the comment pill fades in over
  four placeholder dots that fade away.
- **Craft.** Cells with one or two rounded corners are unions, not paths;
  "quarter discs" are cells with a single corner rounded at 0.75–0.85 × cell.

## Striped wordmark (portrait split field, 3 s, eight-bar letters on black and blue)

- **System.** Bars at pitch 24.4 px, height 12.3 px; 73 bar segments per
  half, curved ends and the notch as row-traced paths. The blue half is not a
  rigid copy — measure both.
- **Motion.** Black half: a 9 % ghost of the mark from frame 0; each bar
  opens horizontally from its own centre while fading ghost → white, 10–30
  frames, on an ease-out with a geometric tail. Blue half: bars scale from
  their left edge with a linear fade, 17–20 frames, cascading top to bottom
  and left to right from frame 4 to 101.
- **Honest finding.** What reads as a digital flicker is the scattered onset
  order; no bar blinks after it settles. When a reference looks random, probe
  it before reproducing randomness.

## What these have in common

One backdrop system, one hero with the only overshoot, one container, one
title, one layer of punctuation; a still first frame; ins that hand off and
outs that run in reverse, faster. Choose those five roles for a brief before
choosing any shape.

## One-shots from short briefs (Fable 5.1, no reference)

Three briefs of one sentence each, judged against the rubric above:

- *"logo reveal for our fintech startup Nimbus. the logo is a cloud. square,
  3 seconds"* — 25/28. A droplet stretches into the cloud's base (band
  opening), lobes billow up in a hand-off, an amber lining draws on. Lost
  points for a backdrop dot field too faint to see and a lining that floated
  off the edge.
- *"make me a year-in-review animation for my coffee shop, ritual coffee.
  portrait, for instagram stories"* — 25/28. A ring-stain lattice, a top-down
  cup as hero, a paper band with three stats cut in and out, beans as
  punctuation, a reverse exit to the title card. Lost points for the cup's
  handle sitting behind the title.
- *"a launch teaser for our note-taking app 'Loop' — should feel premium and
  fun. dark mode"* — 24/28. A note writes itself, the pen draws the loop that
  becomes the mark, the bullets fill on the way back. Lost points for scale:
  the card and lockup filled a fifth of a 1920 × 1080 frame.

The pattern: concept and motion arrive at the bar on the first try; the
misses are visibility (contrast, scale) and collisions (type versus hero).
Check both on a 360 px sheet before timing.
