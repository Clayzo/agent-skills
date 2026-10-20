# Shipping an animation

Getting a finished document into somebody's product. The decision is not
primarily about file size — it is about what the artefact has to do.

## Decision principles
- If the brief mentions the cursor, hover, press, or a value the page sets, the
  answer is a document played by a runtime. Interactivity has no timeline
  representation, and exports do not evaluate bindings at all — see below.
- If the destination is not a page you control — an email, an ad network, a
  slide deck — it is a video. A runtime needs to be loaded from somewhere.
- Pick the renderer once, at integration time, and ship one. Start with WebGL
  for websites, verify coverage over every shipped document, and move to
  CanvasKit only when coverage or an explicit fidelity requirement demands it.
  Do not make the page able to load both.
- **Anything whose rest state depends on a binding does not have a rest state.**
  Exports run neither the interaction graph nor the state machine; they render
  the timeline as authored, states back to back, and ignore bindings entirely.
  A property you authored as `1` and
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
| go anywhere that takes video | `mp4`, or `mov` for alpha (both carry sound) |
| be a still — thumbnail, poster, OG image | `png` |

```bash
clayzo package hero.json hero.clayzo      # document + fonts + images + audio
clayzo export hero.json out.mp4 --format mp4 --tier final
clayzo render-frame hero.json poster.png --tick 300
```

`export` renders on this machine's GPU when that is faster, in a Chrome, Edge,
Brave or Chromium it starts headless: far faster for glass, custom shaders and
other effects. Otherwise it renders on the CPU, on up to eight cores at once.
`--backend cpu` or `--backend gpu` forces one (see `previewing-work.md`).

Video comes from the same renderer as playback, so it is pixel-exact. Runtime
effects, 2.5D projection, and interaction bindings do not survive conversion
to simpler interchange formats; prefer `.clayzo` or rendered media when those
features matter.

## Which player

Two packages, same call, so swapping is one word in an import:

```ts
import { createPlayer } from "@clayzo/webgl-player";      // ~253 KB brotli
import { createPlayer } from "@clayzo/canvaskit-player";  // ~2,535 KB brotli

const player = await createPlayer({ canvas, document });
player.play();
```

In a page, use the component or element below instead of a bare canvas.

**`@clayzo/webgl-player`** when transfer size is the constraint and you know your
documents. Run `checkCoverage` first (`previewing-work.md`), and in CI over
everything you ship.

**`@clayzo/canvaskit-player`** when you cannot enumerate the documents in advance,
when coverage says a document needs it, or when reference-renderer parity is an
explicit requirement. It is identical to what the video export produces. It
loads CanvasKit's wasm from unpkg unless `canvasKitUrl` points at your copy.

If undecided, start with WebGL and make `checkCoverage` a build gate. CanvasKit
is the deliberate fallback for unsupported constructs, design tools with
unbounded user-authored content, and integrations where exact export parity is
worth the substantially larger transfer.

Before shipping, run `clayzo review` and fix its web playback warnings (see
`self-review.md`).

## In a page

`ClayzoPlayer` (React) and `<clayzo-player>` (any page) show a poster, load the
engine only near the viewport, honour reduced motion, label the animation for
screen readers, and never throw into the page. Do not wrap them in your own
matchMedia, IntersectionObserver or poster crossfade.

```bash
clayzo package hero.json hero.clayzo --poster 240   # also writes hero.poster.webp
```

The poster is the frame at that tick, snapped to a frame; the printed JSON
gives the tick to pass as `posterTick`. Pick one where the artwork has
settled: it is also what readers who ask for reduced motion see.

Ship the `.clayzo`: it carries fonts, images and audio. A document JSON given
to the component or `createPlayer` without `loaders` fetches its asset URIs
relative to the page.

React or Next (`@clayzo/canvaskit-player/react` takes the same props):

```tsx
"use client";
import { useEffect, useState } from "react";
import { ClayzoPlayer } from "@clayzo/webgl-player/react";
import { readClayzoBundle, type LoadedClayzo } from "@clayzo/animation/bundle";

export function Hero() {
  const [bundle, setBundle] = useState<LoadedClayzo | null>(null);
  useEffect(() => {
    fetch("/hero.clayzo").then((response) => response.arrayBuffer())
      .then((bytes) => readClayzoBundle(new Uint8Array(bytes))).then(setBundle);
  }, []);
  return (
    <ClayzoPlayer document={bundle?.document ?? null} options={{ loaders: bundle ?? undefined }}
      poster="/hero.poster.webp" posterTick={240} label="A cube folding into the logo" className="hero" />
  );
}
```

| Prop | |
|---|---|
| `document` | The document, or `null` while it loads; the poster shows and sizes the box. |
| `options` | `createPlayer` options other than `canvas` and `document`. |
| `poster`, `posterTick` | An image URL or element, shown until the player draws `posterTick` (default 0). |
| `autoplay` | Plays once revealed. Default `true`. |
| `reducedMotion` | For readers who ask for less motion: `"still"` (default) holds `posterTick`, `"pause"` stops where it is, `"ignore"` plays. Pointer interaction works in all three. |
| `label`, `decorative` | Give one: `label` says what it shows; `decorative` hides it from screen readers. |
| `fallback` | Shown if the player cannot be created, such as without WebGL2; otherwise the poster stays. |
| `onReady` | Gets each new player's `PlayerHandle`, once it shows `posterTick`. |
| `onPlayerError` | Every failure; none is thrown. |

- Inline `document`, `options` and callbacks are safe: re-renders keep the
  running player. It is rebuilt, from the start, only when the document's
  content or a creation option changes; `muted` applies live, and callbacks
  always call the latest function.
- `className`, `style` and other `div` props go on a wrapper div. Size it by
  width; the height follows the document's aspect ratio.
- The entry is marked `"use client"`, so a Server Component can render
  `ClayzoPlayer` with serializable props. Pass functions from a client
  component.

Any other page (WebGL only; import the element once from the page's module
script):

```html
<script type="module">import "@clayzo/webgl-player/element";</script>
<clayzo-player src="/hero.clayzo" poster="/hero.poster.webp" label="A cube folding into the logo"></clayzo-player>
```

- `src` is a `.clayzo` or a document JSON, whose asset URIs resolve against
  it. Attributes: `poster`, `poster-tick` (default: the tick `--poster`
  recorded), `autoplay`, `loop`, `interactive`, `reduced-motion`, `label`,
  `decorative`; `="false"` turns a flag off.
- Events `ready` (`event.detail.player`) and `error` (`event.detail`); methods
  `play()`, `pause()`, `seek()`, `setInput()`, `fire()`; `.player` once ready.
  Content inside the element is the fallback.

## States

A document with a state machine plays its states in both players, the
component and the element, from its initial state.

```ts
const player = await createPlayer({
  canvas, document,
  onStateChange: ({ from, to, cause }) => {},  // after the frame that shows `to`
  onComplete: ({ state }) => {},               // a `once` state reached its last frame
});
player.setInput("progress", 0.4);   // boolean and number inputs: conditions and scrub states
player.fire("reset");               // trigger inputs: `input:reset` transitions
player.setState("done");            // plays a transition's via on the way; { instant: true } jumps
player.currentState;                // { id, name }; `player.states` lists them all
```

- The state decides what loops; `loop` does not apply. `stateMachine: false`
  plays the plain timeline (a thumbnail); `initialState` starts elsewhere.
- Pointer transitions need `interactive`, on by default. `onPointer` reports
  enter, leave, down, up and click on the canvas and on hit areas.
- `setInput` and `fire` throw for an input the document does not declare, and
  `setState` for a state it does not have. Number inputs clamp to their
  declared range. A callback that throws goes to `onError`.
- With the component, pass these callbacks in `options` and drive the handle
  `onReady` gives you; `<clayzo-player>` has `setInput()` and `fire()`, and
  `.player` for the rest.

## Sound

A document's audio nodes play in both players and export with the video.

- **Export:** mp4, webm and mov include the sound; other formats are silent.
  `--no-audio` leaves it out. If an export says its ffmpeg can't export sound,
  run `clayzo update`.
- **Players start muted**, and browsers allow sound only after a click or key
  press. Show a sound control when `player.hasAudio` and call
  `player.setMuted(false)` from it (`element.player.setMuted(false)` for
  `<clayzo-player>`). If you pass your own `loaders`, include
  `audioAssetLoader`; a `.clayzo` carries its audio. The iOS player:
  `isMuted = false`.

## Fonts

The thing neither package solves. A document referencing a family by name needs
those bytes at play time, and if no loader supplies them the text draws
*nothing* — it does not substitute and it does not warn.

Give every text run a `fontAssetId`: `@clayzo/webgl-player` draws text only in
a face the document carries as a `font` asset, so a run that names only a
family is not drawn there, packaged or not. Packaging as `.clayzo` embeds the
fonts and ends the problem. Otherwise the host must wire a font loader, and
`@clayzo/canvaskit-player`'s `onPrewarm` reports `unresolvedFamilies` so this
is caught in development rather than in production.
