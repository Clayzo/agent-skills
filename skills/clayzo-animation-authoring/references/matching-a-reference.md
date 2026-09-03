# Matching a reference

Recreating a clip one to one, or checking a piece against a mockup or a
brand's existing motion. Looking is not enough; measure, then compare the
same way you measured.

## Workflow

1. **Probe the clip.** `ffprobe` for size, frame rate and count. Extract a
   contact sheet at 6–8 fps and single frames at the beats you can see.
2. **Sample colours** from a settled frame with a 1 × 1 crop dumped as raw
   RGB — the background, every fill, and any tint on white.
3. **Find geometry with connected components.** Classify pixels by nearest
   palette colour, flood-fill, read centroids and bounding boxes. Row
   profiles (x-extent per row) tell you a shape's corners: after a 45°
   rotation a square corner reaches 0.707 × cell from the centre along an
   axis and a rounded one 0.5 ×. Same-colour shapes that touch merge into one
   component; split them by their extreme points.
4. **Fit the lattice** from centroids by least squares on a few clean
   circles. Then predict where every other unit must be and check the
   prediction, including the leftover dots a reveal leaves in its last frame
   — they are the origins.
5. **Timeline everything.** Dump the clip at 30–60 fps at half resolution and
   probe per frame: a radius, a colour channel at a point, the leftmost
   coloured pixel on a row, the fraction of a window that is a colour. Write
   the curve down in seconds before choosing an easing. Probe a window's
   maximum rather than a pixel where compression lifts small features.
6. **Compare, do not admire.** Side by side (`hstack`), difference
   (`blend=all_mode=difference,negate`), and tiled timelines of the same crop
   from both clips (`fps=30,crop=…,tile=CxR`) with the reference row above
   yours. Blob the same frames from both and print the centroid deltas.
7. **Change one thing, regenerate, re-tile.** A model that fits the settled
   frame but not the first 0.3 s is the wrong model, not a timing error.

## Reading curves

- A value at 30–45 % of its travel one frame after it starts, then easing,
  is a steep ease-out, `{ 0.1–0.2, 0.6–0.9, 0.3–0.5, 1 }`.
- A rise that overshoots and returns is a one-bounce settle. Note the peak
  time and the settle time separately; they are rarely symmetric.
- A plateau before a snap back is not a spring. Model it as three segments.
- Per-frame linear keys are legitimate when the shape of a curve matters more
  than its smoothness — band widths, stripe cascades, chain zooms.
- Whole-pattern motions leave signatures: a staircase whose step *and* square
  both shrink is one scaled group; squares that shrink about their own
  centres keep their spacing.

## Traps

- Video colour is not source colour; match the clip.
- Occlusion hides structure. Probe before the hero arrives, or at points
  outside it, before deciding a chain has gaps.
- A clip's first two or three frames are usually a still. Keep them.
- Playback frame rate and authored rate differ; author at the clip's rate so
  keyframes land on its frames.
- Scale probes saturate: a probe window of ±44 px reads "full" at 70 % size
  if the shape's diagonal is what you are measuring.
