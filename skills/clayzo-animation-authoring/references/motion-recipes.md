# Motion recipes

Entrance and settle shapes measured from shipped references, written as
tracks at 240 ticks per second. `motion-fundamentals.md` says what to aim
for; this says what the numbers were when it worked. Reach for a named
primitive before inventing a curve, and use at least three different ones in
any piece longer than a second.

## Primitives

| Name | Track | Ease | Duration | Where it came from |
|---|---|---|---|---|
| Pop | scale 0 → 1 | `{0.1, 0.9, 0.3, 1}` | 0.10 s | discs, ring dots |
| Pop with landing | scale 0 → 1.06 → 1 | out, then in-out | 0.07 s + 0.05 s | small dots |
| Flood | clip Ø 0 → full | `{0.2, 0.45, 0.45, 1}` | 0.17–0.23 s | corner reveals |
| Drain | clip Ø full → 0 | `{0.5, 0, 0.7, 0.7}` | 0.12–0.15 s | reverse reveals |
| Hero pop, plateau, snap | scale 0 → 1.18 (0.28 s) → 1.19 (0.12 s) → 1.0 (0.10 s), opacity 0 → 1 in the first 0.15 s | out / linear / in-out | 0.50 s | starburst |
| One-bounce rise | scale 0.4 → 1.0 (0.23 s) → 1.15 (0.40 s) → 1.0 (0.15 s), opacity 0 → 1 in the first 0.17 s | out / soft out / in-out | 0.78 s | large numerals behind a title |
| Fade and settle | opacity 0 → 1 (0.34 s) with scale 1.12 → 1.0 (0.27 s), anchored at the baseline | `{0.25, 0.5, 0.45, 1}` | 0.34 s | title text |
| Zoom from the axis | group scale 0 → 1 about the point where it crosses the frame's axis; opacity 0 → 1 over the same span | measured, played as linear segments | 0.27 s centre, 0.43 s neighbours, 0.50 s outer | pixel chains |
| Cascade | per-bar opacity ramps top to bottom, each 10–30 frames, onsets 2–4 frames apart | out | 0.5–1.7 s | striped wordmark |
| Exponential grow-fade | `v = v0 + (1 − v0)(1 − e^(−t/τ))`, τ ≈ 0.3 × duration, scale and opacity together | bezier fit to that curve | 16–40 frames | pattern cells in a card |
| Spin-in | rotation 155° clockwise while scaling from 0, settling with a point on an axis | out | 1.5 s | badge starburst |
| Slide-in | position −14 px → 0 with opacity | out | 0.2 s | placeholder text bars |

## Orders that read as designed

- **Sequential hand-off** — each element starts as the previous lands. For a
  handful of large elements.
- **Wave with two rates** — 50 ms per column, 100 ms per row, starting at the
  corner nearest the previous beat (bottom-right → top-left). For grids.
- **Centre out, lagging by distance** — the central unit first and fastest,
  neighbours ~50 ms later at the same pace, the outer units starting almost as
  early but growing at half the speed. For stripes and anything radiating.
- **Measured irregular order** for rings and clusters — bottom row left to
  right, then the right column upward, then the top row right to left, then
  the left column, 50–70 ms apart. Never a clean clockwise sweep.
- **Reverse order to exit**, with a shorter stagger (36 ticks out against
  48–56 in) and a faster collapse. Outs are about 30 % quicker than ins.

## Beats of a three-to-six-second piece

1. 0.0–0.1 s — a still frame. The resting state must read on its own.
2. 0.1–0.5 s — the backdrop system arrives (zoom, cascade, tiles).
3. 0.4–0.7 s — the hero pops, with the piece's one overshoot.
4. 0.6–1.1 s — containers and the title: fade and settle, not slide.
5. 1.0–1.8 s — secondary content (numerals, subtitles), one bounce.
6. 1.3–2.0 s — punctuation (dots, badges), irregular order.
7. Hold to the end, or drain in reverse so the piece can cut to itself.

## Anti-patterns

- The same ease on every track. The reference pieces use five distinct shapes
  inside two seconds.
- Overshoot everywhere. One hero overshoots; everything else pops or fades.
- Symmetric in and out timing.
- Motion before the first frame has been still for at least two frames.
- Starting a hero before its backdrop has settled to 90 %.
