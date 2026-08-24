# Shipping an animation

Getting a finished document into somebody's product. The decision is not
primarily about file size — it is about what the artefact has to do.

## Decision principles
- If the brief mentions the cursor, hover, press, or a value the page sets, the
  answer is a document played by a runtime. Interactivity has no timeline
  representation, and exports do not evaluate bindings at all — see below.
- If the destination is not a page you control — an email, an ad network, a
  slide deck — it is a video. A runtime needs to be loaded from somewhere.
- Pick the renderer once, at integration time, and ship one. Do not make the
  page able to load both.
- **Anything whose rest state depends on a binding does not have a rest state.**
  Exports do not run the interaction graph; they render the timeline as
  authored and ignore bindings entirely. A property you authored as `1` and
  bound to `multiply` by a proximity value exports at `1` — full strength —
  not at the zero its resting evaluation would give. Author the resting pose
  into the document and let bindings `add` to it, rather than authoring a
  maximum and multiplying it down.
- An interactive document must still read correctly with the pointer away.
  That is what every still, every video export, and every first paint shows.

## Choosing an output

| The result must… | Produce |
|---|---|
| respond to a cursor, or expose inputs a page can set | `.clayzo` + a player |
| play in a page you control, without interactivity | the document + a player |
| go anywhere that takes video | `mp4`, or `mov` for alpha |
| be a still — thumbnail, poster, OG image | `png` |

```bash
clayzo package hero.json hero.clayzo      # document + fonts + images
clayzo export hero.json out.mp4 --format mp4 --tier final
clayzo render-frame hero.json poster.png --tick 300
```

Video comes from the same renderer as playback, so it is pixel-exact. Runtime
effects, 2.5D projection, and interaction bindings do not survive conversion
to simpler interchange formats; prefer `.clayzo` or rendered media when those
features matter.

## Which player

Two packages, same call, so swapping is one word in an import:

```ts
import { createPlayer } from "@clayzo/webgl";      // ~253 KB brotli
import { createPlayer } from "@clayzo/canvaskit";  // ~2,535 KB brotli

const player = await createPlayer({ canvas, document });
player.play();
```

**`@clayzo/webgl`** when transfer size is the constraint and you know your
documents. Run `checkCoverage` first, and in CI over everything you ship.

**`@clayzo/canvaskit`** when you cannot enumerate the documents in advance, or
when coverage says a document needs it. It is the reference: correct by
construction, and identical to what the video export produces.

If undecided, ship CanvasKit. Page weight is a problem you can measure and fix
later; a construct that silently does not draw is a bug users report and you
cannot reproduce.

## Fonts

The thing neither package solves. A document referencing a family by name needs
those bytes at play time, and if no loader supplies them the text draws
*nothing* — it does not substitute and it does not warn.

Packaging as `.clayzo` embeds them and ends the problem. Otherwise the host
must wire a font loader, and `@clayzo/canvaskit`'s `onPrewarm` reports
`unresolvedFamilies` so this is caught in development rather than in production.
