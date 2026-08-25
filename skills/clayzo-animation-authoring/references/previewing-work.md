# Previewing what you authored

Closing the loop between writing a script and knowing whether it worked. An
agent that writes an animation and never looks at it is guessing, and the
guesses are usually wrong in ways validation cannot catch: a layer behind
another, a keyframe that never moves, an effect washing out its own input.

## Decision principles
- Look at several frames, not one. A single frame answers "what is at tick
  152"; the real question is "does this animate the way I meant", and that
  needs the timeline sampled.
- Look after every meaningful change, not once at the end. A wrong assumption
  costs one edit to fix when you catch it immediately and a rewrite when you
  catch it after four more.
- Read the diagnostics the CLI prints. A frame that renders is not a frame that
  rendered *correctly* — missing fonts and approximated effects come back as
  warnings while the PNG still looks plausible.
- A blank or near-blank frame is a bug, not a boring moment. Check it before
  moving on.

## Rendering frames

```bash
clayzo render-frame hero.json frame.png --tick 300
```

One frame at one tick. `--tick` is in ticks, not frames or seconds.

```bash
clayzo render-frame hero.json sheet.png --samples 6 --max-dimension 320
```

Six ticks spread evenly across the document, laid out as one labelled sheet.
**This is the one to reach for by default.** It costs about the same as a single
frame — the expensive part is starting CanvasKit, not drawing — and it shows
whether anything actually moves.

`--max-dimension` bounds the longest side. Use it: a 1920×1080 document sampled
six times is a large image to look at, and 320 is enough to see composition and
motion.

Both commands print JSON to stdout with the output path, the ticks rendered,
dimensions, and a diagnostics count.

## The preview server

```bash
clayzo preview hero.json
```

Serves one page: the document playing, scrubable, with an export panel. Open
the URL it prints. Space plays, arrow keys step a frame, dragging the timeline
scrubs, and shift-dragging it sets an export range.

**Export from that page is dramatically faster on anything with glass, a
custom shader, or any other runtime effect.** Frames rasterize on the machine's
GPU rather than being interpreted per pixel in Node — measured on the
nine-glass-shard section of a 1920x1080 document, **36 ms a frame against
53,000 ms**. Everything after the frame is unchanged: the same ffmpeg, the same
quality tiers, the same verification.

Two things worth knowing. GPU and CPU Skia are not bit-identical — about 0.16
mean channel on this corpus — so a browser export matches the preview you were
looking at rather than what a terminal export produces. And the browser has to
stay open, because it is doing the work.

**Stop it when you are done.** It holds a port until it is killed, and it
prints its own pid on startup for exactly that:

```
clayzo preview  http://127.0.0.1:53219/
pid 85619 · ctrl-c, or: kill 85619
```

Use the pid. A broad process-name pattern is a guess and may stop an unrelated
process or miss the preview entirely.

It also stops itself after ten minutes with no browser attached, so a forgotten
one does not outlive the session — but an agent that started it should not rely
on that.

An agent that wants the speed without a person clicking anything can use
`clayzo export hero.json out.mp4 --format mp4 --backend browser`, which starts
the same server, prints its URL, and waits for a browser to connect.

## The loop

```bash
npx tsx author.ts                                                  # write the document
clayzo render-frame out/hero.json sheet.png --samples 6 --max-dimension 320
```

Then open the PNG. Adjust the script, run both again. Validation catches
structural mistakes; only looking catches design ones.

## Loops

```bash
clayzo render-frame loop.json seam.png --seam
```

The one measurement a loop turns on: how far the last displayed frame is from
the first, reported against an ordinary frame step so the number means
something. See `loop-design.md`.

## Which renderer will play it

If the project ships `@clayzo/webgl-player`, confirm it can draw what you authored —
mattes, nested clips and colour-space intrinsics in custom shaders need
`@clayzo/canvaskit-player`:

```ts
import { checkCoverage } from "@clayzo/webgl-player";
const coverage = await checkCoverage(document);
```

An unsupported construct is not an error at play time. The node is simply not
drawn, and nobody finds out until a user does.

## Deeper checks

`clayzo review hero.json packet.json` renders semantic samples across
several backgrounds and adds diagnostics for blank frames, clipping, contrast
and motion. Heavier than `render-frame --samples`; worth it before shipping,
not on every edit.
