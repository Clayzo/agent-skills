# Authoring scripts

Writing a program that emits an animation document. This is the mechanics; the
other skills in this directory are about what to build, not how.

## Decision principles
- Write TypeScript against the types, never JSON by hand. The shapes are
  unobvious enough to get wrong, and the failure is quiet: a document can be
  structurally wrong and still render *something*, so inspecting the output
  does not tell you it is broken.
- Import from `@clayzo/animation-engine/authoring`. It carries the node types,
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
} from "@clayzo/animation-engine/authoring";

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
| `fill: { color }` | `style: { fill: { color, opacity } }` |
| `{ segments: [{ from, to }] }` | `{ kind: "keyframed", keyframes: [...] }` |
| a number parameter with just a value | also needs `minimum` and `maximum` |
| a partial transform | all five of position, anchor, scale, rotation, skew |

## Interactivity

`withInteraction(scene, contributions)` layers pointer behaviour onto a
finished scene. Reach for a macro before writing a binding graph by hand — see
`pointer-interaction.md` for which one and why.

## Packaging

`packageClayzoBundle(document, { assetBytes })` produces a `.clayzo`: the
document with its fonts and images embedded. It refuses to build one whose text
would render as nothing, which is the last cheap moment to catch a missing font.
