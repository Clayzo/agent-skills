# Product microinteractions

## Decision principles
- Confirm input immediately, preserve spatial continuity, and reserve larger motion for meaningful state changes.
- Make enter, update, success, error, and exit states distinguishable without relying on motion alone.
- Keep interruption safe: new input should retarget rather than queue stale animations.

## States
Discrete states (idle, hover, pressed, loading, done) are a state machine over
markers, and both web players run it. Do not write one in host code. For
responses that follow the pointer continuously, use bindings
(`pointer-interaction.md`); the two share inputs and can run together.

```ts
import { assertValidAnimationDocument, stateMachine, withStateMachine } from "@clayzo/animation/authoring";

// scene's markers: idle, lift (a via), hover (zero length), pressed, loading, done
const machine = stateMachine({ initial: "idle" })
  .state("idle", { marker: "idle" })                        // the initial state loops
  .state("hover", { marker: "hover", playback: "once" })
  .state("pressed", { marker: "pressed", playback: "once" })
  .state("loading", { marker: "loading", playback: "scrub", scrubInput: "progress" })
  .state("done", { marker: "done", playback: "once" })
  .on("pointerenter", "idle", "hover", { hitArea: "button", viaMarker: "lift" })
  .on("pointerleave", "hover", "idle", { hitArea: "button" })
  .on("pointerleave", "pressed", "idle", { hitArea: "button" })
  .on("pointerdown", "hover", "pressed", { hitArea: "button" })
  .on("pointerup", "pressed", "loading", { hitArea: "button" })
  .when("loading", "done", { input: "progress", operator: "gte", value: 1 })
  .on("input:reset", "done", "idle");

const document = withStateMachine(scene, machine, {
  inputs: { progress: { kind: "number", minimum: 0, maximum: 1, defaultValue: 0 }, reset: "trigger" },
  hitAreas: { button: { nodeId: "button", padding: 8 } },
});
assertValidAnimationDocument(document);
```

The host sets `progress` as the work runs and fires `reset` afterwards.
`withStateMachine` stamps the revision.

- A state plays from its marker's tick to the marker's end; `exitMarker` runs
  it to the end of a later marker. A zero-length marker holds one frame.
- `playback`: `loop` (the default for the initial state, or a marker with
  `payload: { loop: true }`); `once` (every other state's default: plays,
  holds its last frame, fires `complete`);
  `ping-pong`; `scrub`, which follows a number input, its minimum on the first
  frame and its maximum on the last.
- Triggers: `pointerenter`, `pointerleave`, `pointerdown`, `pointerup`,
  `click`, `complete`, `input:<name>` for a declared trigger input, and
  `always`, written `.when(from, to, conditions)` and taken whenever its
  conditions hold.
- `hitArea` limits a pointer trigger to one declared hit area; without it the
  trigger fires for the whole canvas.
- Conditions compare a declared boolean (`eq`, `ne`) or number input (`eq`,
  `ne`, `lt`, `lte`, `gt`, `gte`) with a constant; all must hold.
- `"*"` as `from` means any other state. A transition from the current state
  beats one from `"*"`, then the first declared wins. An `always` from `"*"`
  pulls every other state into its target while its condition holds, so name
  the state it leaves from.
- `viaMarker` plays that marker's ticks on the way in, already in the new
  state. Going back mid-via reverses it from where it is, so author vias that
  read well backwards.
- Under reduced motion the players show each state at its last frame (vias
  and `once` states jump to their end, `complete` still fires), so make every
  state's last frame read on its own. Touch screens have no hover: a touch
  enters and presses at once.
- Exports and `review` play the plain timeline, states back to back. Set
  `design.tokens.looping: false`; if review flags a boundary between states
  that no transition plays across, mark it as a cut (`self-review.md`).

Errors throw from `assertValidAnimationDocument`; warnings (w) are in
`validateAnimationDocument(document).diagnostics`.

| Code | Means |
|---|---|
| `state_machine.schema` | A field or value the format lacks, such as a misspelt `playback`. |
| `reference.state_marker`, `reference.initial_state`, `reference.transition_state` | A marker or state that does not exist. |
| `state_machine.segment`, `transition.segment` | An exit marker ending before its entry starts; a via outside the document. |
| `state_machine.scrub` | A `scrub` state without a declared number `scrubInput`, or `scrubInput` on another playback. |
| `state_machine.trigger_input` | `input:<name>` with no declared trigger input of that name. |
| `state_machine.hit_area` | `hitArea` on a trigger that is not a pointer event, or an undeclared hit area. |
| `state_machine.condition` | An undeclared input; a boolean compared with a number or with `lt`…; a trigger, point or colour input. |
| `state_machine.always_unconditional`, `state_machine.always_self` | An `always` with no condition, or into its own state. |
| `state_machine.trigger_unknown` (w) | A trigger nothing fires. |
| `state_machine.complete_never_fires` (w) | `complete` from a state that does not play `once`. |
| `state_machine.unreachable_state` (w) | Nothing leads to the state; only `setState` enters it. |
| `state_machine.condition_cycle` (w) | `always` transitions that can all hold at once chase each other. |
| `state_machine.duplicate_name` (w) | Two states share a name; `setState(name)` finds the first. |

## Diagnostics to inspect
- First response tick, landing tick, contrast range, flashes, frozen intervals, clipped focus/labels, and reduced-motion alternatives.

## Anti-patterns
- Delayed acknowledgement, celebratory motion on routine actions, hover-only meaning, repeated bounce, or animated layout shift.

## Parameter ranges
- Press feedback: 60–120 ms and scale `0.96–0.99`.
- Tooltip/menu entry: 100–220 ms. Success emphasis: 250–500 ms with at most one overshoot.

## Adaptation
Tune amplitude to action frequency and risk. Frequent controls need quieter motion; destructive or asynchronous actions need clearer state persistence, not more spectacle.
