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

Six frames spread evenly across the document, laid out as one labelled sheet.
**This is the one to reach for by default.** It costs about the same as a single
frame — the expensive part is starting CanvasKit, not drawing — and it shows
whether anything actually moves. `--start` and `--end` (ticks) spread the
frames over one span instead: `--samples 6 --start 0 --end 168` is the
opening 0.7 s at 240 ticks per second.

`--max-dimension` bounds the longer side of each frame, not of the sheet. Use
it: a 1920×1080 document sampled six times is a large image to look at, and
320 is enough to see composition and motion.

Both commands print JSON to stdout with the output path, the ticks rendered,
dimensions, and a diagnostics count.

## Signing in

`render-frame`, `review`, `package`, `export` and `preview` need a signed-in
Clayzo account: `clayzo login` on the person's machine, or `CLAYZO_TOKEN` in
CI. `clayzo doctor` says whether this machine is signed in. Where the CLI can't reach Clayzo (it says `CLAYZO_OFFLINE`), such as
Claude's sandbox, call the Clayzo connector's `get_render_pass` and put the
pass in `CLAYZO_TOKEN` for each command:

```bash
CLAYZO_TOKEN='clayzo_pass_…' clayzo render-frame hero.json sheet.png --samples 6 --max-dimension 320
```

A pass lasts an hour; get a new one when the CLI says it expired. If the
connector isn't connected, ask the person to connect Clayzo rather than
rendering some other way. Nobody can open `clayzo preview` from a sandbox, so
deliver with `clayzo export`.

## The preview server

```bash
clayzo preview hero.json
```

Serves one page: the document playing, scrubable, with an export panel. Open
the URL it prints. Space plays, arrow keys step a frame, dragging the timeline
scrubs, and shift-dragging it sets an export range.

Shapes, paths and text draw fast on the CPU: a flat 3-second piece exports in
about a second. **Glass, custom shaders and other runtime effects are
dramatically faster on the GPU**, where this page draws them — measured on the
nine-glass-shard section of a 1920x1080 document, **36 ms a frame against
53,000 ms**. Everything after the frame is unchanged: the same ffmpeg, the same
quality tiers, the same verification.

Two things worth knowing. GPU and CPU Skia are not bit-identical — about 0.16
mean channel on this corpus — so a GPU export matches the preview you were
looking at rather than a CPU export. And the browser has to
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

`clayzo export`, `render-frame` and `review` get the same speed with nobody
clicking anything. When a document draws faster on the GPU, they start a
Chrome, Edge, Brave or Chromium installed on the machine, headless, and draw
there; with no such browser or no hardware GPU they draw on the CPU. A `[render]` line on stderr names the GPU when
one draws. `--backend cpu` forces the CPU; `--backend browser` waits for a
person to open the page, so an agent should not use it.

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
mattes, nested clips, Lottie nodes and colour-space intrinsics in custom
shaders need `@clayzo/canvaskit-player`:

```ts
import { checkCoverage } from "@clayzo/webgl-player";
const coverage = await checkCoverage(document, loaders); // { supported, reasons }
```

Pass the loaders the player gets (a bundle's included): text in a face they
don't supply is reported as not encoded. It compiles custom shaders to check
them, so run it in a browser for a document that has one. An unsupported construct is not an error at play time.
The node is simply not drawn, and nobody finds out until a user does.

## Deeper checks

`clayzo review hero.json review/` checks every frame for parts that pop,
shake, collide or run off the canvas, checks text contrast, the loop point and
web playback cost, and writes a contact sheet. Heavier than
`render-frame --samples`; run it before delivering, not on every edit. Its
codes and fixes are in `self-review.md`.

## What the terminal actually tells you

- `render-frame` and `export` print diagnostic counts, and the codes behind
  them when any fired; drawing text is not counted. The messages are in
  `review.json`.
- `--samples` lands on displayed frames; `--tick` renders any tick. Twelve
  `render-frame` calls in parallel are cheap.
- Export cost is per *layer*, not per second. A flat document (rects,
  ellipses, text, a few clips) renders at 0.05–0.4 s per 1080p frame on the
  CPU; every node with `opacity` below 1, every isolated group, every drop
  shadow and every effect adds an offscreen surface per frame, and that is
  how a story of sixty translucent stains takes seventeen minutes. Progress
  is printed to stderr about once a second, with an ETA once a few frames are
  done; if the CPU is drawing and that ETA exceeds a minute, export from
  `clayzo preview`.
- `export` writes a `<output>.preview.png` sheet beside the file: up to 20
  frames, 320 px wide, spread across the whole export. It is a convenience,
  not a deliverable.
