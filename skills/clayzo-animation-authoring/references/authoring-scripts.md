# Authoring scripts

Writing a program that emits an animation document. This is the mechanics; the
other skills in this directory are about what to build, not how.

If the project is not initialized yet, run `npx clayzo@latest init`. The
initializer installs the typed authoring package, the lightweight WebGL player,
the local CLI (including the CanvasKit preview/export dependency), and the
Clayzo skills. Add the CanvasKit website player only when the integration
actually needs full-fidelity embedded playback.

## Decision principles
- Write TypeScript against the types, never JSON by hand. The shapes are
  unobvious enough to get wrong, and the failure is quiet: a document can be
  structurally wrong and still render *something*, so inspecting the output
  does not tell you it is broken.
- Import from `@clayzo/animation/authoring`. It carries the node types,
  the value types, `constant`, validation, revision stamping, interaction
  authoring with all eight pointer macros, and bundling. Other entry points
  exist and are machinery.
- Build a document, do not mutate one. Compose helpers that return nodes, then
  assemble at the end. Scripts that mutate a shared object are the ones that
  produce documents nobody can reason about.
- Validate before writing the file, every time. `assertValidAnimationDocument`
  names the node and the field.
- Stamp the revision with `withComputedRevision`. A document's identity is the
  hash of its canonical form, so an unstamped one declares a revision that does
  not match its contents.

## The shape of a script

```ts
import {
  ANIMATION_DOCUMENT_VERSION, assertValidAnimationDocument, constant,
  withComputedRevision,
  type AnimationDocument, type SceneNode, type Transform25D,
} from "@clayzo/animation/authoring";

// 1. Helpers that return nodes. `transform` first — every node needs one and
//    there are no partial transforms.
const transform = (x: number, y: number): Transform25D => ({
  position: constant({ x, y, z: 0 }),
  anchor: constant({ x: 0, y: 0, z: 0 }),
  scale: constant({ x: 1, y: 1, z: 1 }),
  rotation: constant({ x: 0, y: 0, z: 0 }),
  skew: constant({ x: 0, y: 0 }),
});

// 2. A `base` helper for the fields every node repeats.
const base = (id: string, x: number, y: number) => ({
  id, name: id, visible: true, inTick: 0, outTick: DURATION,
  opacity: constant(1), transform: transform(x, y), blendMode: "normal" as const,
});

// 3. Assemble, stamp, validate, write.
const document = withComputedRevision({
  version: ANIMATION_DOCUMENT_VERSION, id: "hero", revision: "", name: "Hero",
  canvas: { width: 1920, height: 1080, pixelAspectRatio: 1, backgroundColor: BG },
  timing: { ticksPerSecond: 240, frameRate: { numerator: 30, denominator: 1 },
            durationTicks: DURATION, displayStartTick: 0 },
  assets: {}, compositions: { main: composition }, rootCompositionId: "main",
  components: {}, markers: [],
  capabilities: { required: [], optional: [], unsupportedPolicy: "warn" },
});
assertValidAnimationDocument(document);
```

Nodes live in a map keyed by id, and `rootNodeIds` lists what draws at the top
level, in back-to-front order. A node not reachable from `rootNodeIds` is not
drawn — which is how a node referenced only as a clip path or a matte source
stays out of the frame.

## Ticks, not seconds

`ticksPerSecond` is the document's time base (240 is conventional) and
`frameRate` is what it plays back at. Every `inTick`, `outTick` and keyframe
`tick` is in ticks. At 240 ticks and 30 fps, one frame is 8 ticks — so
keyframes off that grid land between frames and never display exactly.

## Animatables

Every animated property is one of two shapes:

```ts
constant(value)
{ kind: "keyframed", keyframes: [{ id, tick, value, interpolation }] }
```

`interpolation` is `"hold" | "linear" | "bezier" | "spring"`. `bezier` needs
`easing: { x1, y1, x2, y2 }`; `spring` needs `spring: { mass, stiffness,
damping }`. The last keyframe should be `"hold"` — there is nothing after it to
interpolate toward.

## Effect parameters bind by name

An effect's `parameters` array is looked up by name, not by position, and a
name that does not match is not an error — the effect uses its default and
carries on. A misspelled or invented parameter validates, renders, and does
nothing. Order is irrelevant.

`effect-parameters.md` in this directory lists every parameter each effect
reads, with its type, default and clamp range. It is generated from the
renderer, so it cannot drift from what the code does. Read it rather than
guessing a name.

`custom-sksl` is the exception and the only place position matters: its
uniforms bind against the flattened `parameters` array in declaration order —
`number` and `boolean` take one float, `point` two, `color` four — so the
shader's uniform order and that array have to agree.

`number` and `point` parameters also require `minimum` and `maximum` on the
node. Omitting them is a validation error, and it is the most common one.

An effect's `inputId` names the node it draws; that node must **not** be listed
in `rootNodeIds`, or it is drawn twice. It may name another *effect* node, and
chaining is how compound looks are built — blur into a threshold into a shadow
is the whole gooey technique. And `glass` and `refraction` sample the
surface as already drawn, so the effect has to come *later* in `rootNodeIds`
than whatever it should refract.

## Common mistakes

| Looks right | Actually |
|---|---|
| `childIds` on a group | `children` |
| `radiusX` / `radiusY` on an ellipse | `size: constant({ width, height })` |
| `fill: { color }` | `style: { fill: { color: constant(c), opacity: constant(1) } }` — every colour is an animatable |
| `{ segments: [{ from, to }] }` | `{ kind: "keyframed", keyframes: [...] }` |
| a number parameter with just a value | also needs `minimum` and `maximum` |
| a partial transform | all five of position, anchor, scale, rotation, skew |

## Facts that bite

Each of these produced a document that validated and rendered something
plausible before it was noticed.

- Frames are transparent wherever nothing is drawn; `canvas.backgroundColor`
  is metadata. Draw a full-canvas `rect` as the first root node, or exports
  come out on black.
- A keyframed track's first value holds before its first keyframe. A numeral
  that scales from 0.5 at 1.0 s is sitting there at half size from tick 0
  unless its `inTick` is the cue.
- A `composition` needs `width`, `height` and `durationTicks` as well as
  `nodes` and `rootNodeIds`.
- `rect` and `ellipse` draw centred on the node's position; `path` vertices
  are in the node's local space. The transform is `(point − anchor) × scale`,
  then rotation (`rotation.z` in degrees, clockwise on screen), then
  `+ position`.
- Overlapping children of a group at partial opacity double up at the
  overlaps unless the group has `isolation: true`.
- `polystar` puts its first outer tip at `rotationOffset − 90°`.
- SVG arc commands are flattened to straight lines on import; build curves
  from ellipses or cubic tangents.
- Write the document with `writeFileSync`. A top-level `await` fails under
  `tsx` in a CommonJS project, and the failure names esbuild, not your script.
- A keyframe's `interpolation` and `easing` describe the segment *leaving* it,
  toward the next keyframe; the last keyframe holds. A `bezier` keyframe with
  no `easing` is an error at render time, not a silent linear.
- Keyframe ids must be stable ids and unique; generate them from the node id,
  the property and the index so a document with a thousand keyframes never
  collides.
- Transforms compose: a child's transform is applied in its parent's local
  space, so a lattice group can drift while every unit pops about its own
  centre.
- Markers are sample hints for `review`, but keyframe-dense documents drown
  them; verify beats with your own `render-frame --samples` sheets.
- Node `opacity` below 1 allocates an offscreen layer for that node on every
  frame. Six hundred fading dots is six hundred layers a frame and a
  ten-minute export. Animate `style.fill.opacity` (and `stroke.opacity`) on
  solid shapes instead, and merge a field of units into a few `path` nodes;
  reserve node `opacity` for groups that must fade as one (with `isolation`).
- `PathVertex.inTangent` and `outTangent` are offsets from the vertex's own
  `point`, not absolute positions.
- Colour channels are 0–1 floats, not 0–255.
- Only a *clip* node crashes at scale 0 (its transform is inverted). Ordinary
  nodes and groups at scale 0 simply draw nothing.
- Nodes that never change are cached as static pictures and cost nothing per
  frame; keep static geometry in its own nodes rather than folding it into
  animated ones.
- Text `wrap` defaults to wrapping inside the box; set `wrap: "none"` on
  single lines. A glyph the face lacks draws as a hollow box with no warning
  (the "№" and "·" family are the usual casualties); keep to ASCII unless the
  face is known to carry the character.
- `EffectNode.capability` is required and unvalidated: `{ requires:
  ["image-filter"], fallback: "passthrough", maxTemporarySurfaces: 1 }` for
  filters, `["runtime-effect"]` for shader effects.
- Author at the frame rate the piece will play at (60 fps is 4 ticks per
  frame at 240 tps) and snap every keyframe to it.

## Interactivity

`withInteraction(scene, contributions)` layers pointer behaviour onto a
finished scene. Reach for a macro before writing a binding graph by hand — see
`pointer-interaction.md` for which one and why.

## Packaging

`packageClayzoBundle(document, { assetBytes })` produces a `.clayzo`: the
document with its fonts and images embedded. It refuses to build one whose text
would render as nothing, which is the last cheap moment to catch a missing font.
