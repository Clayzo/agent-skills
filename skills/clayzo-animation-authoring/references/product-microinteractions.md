# Product microinteractions

## Decision principles
- Confirm input immediately, preserve spatial continuity, and reserve larger motion for meaningful state changes.
- Make enter, update, success, error, and exit states distinguishable without relying on motion alone.
- Keep interruption safe: new input should retarget rather than queue stale animations.

## States
- Give each state its own marker (idle, hover, pressed, loading, done) and declare the machine with `stateMachine({ initial }).state(…).on(…).when(…)` and `withStateMachine(document, machine, { inputs, hitAreas })`. Do not write a state machine in host code.
- Loop the resting state. Play transient states `once`: they hold their last frame and fire `complete`. Use `playback: "scrub"` with a number input for progress.
- Pointer transitions take `hitArea` to listen on one shape. `when(from, to, { input, operator, value })` lets the host drive states with `setInput` (`loading eq true`, `progress gte 1`).
- `viaMarker` plays a segment on the way in, and leaving partway reverses it, so author vias that also read well backwards.

## Diagnostics to inspect
- First response tick, landing tick, contrast range, flashes, frozen intervals, clipped focus/labels, and reduced-motion alternatives.

## Anti-patterns
- Delayed acknowledgement, celebratory motion on routine actions, hover-only meaning, repeated bounce, or animated layout shift.

## Parameter ranges
- Press feedback: 60–120 ms and scale `0.96–0.99`.
- Tooltip/menu entry: 100–220 ms. Success emphasis: 250–500 ms with at most one overshoot.

## Adaptation
Tune amplitude to action frequency and risk. Frequent controls need quieter motion; destructive or asynchronous actions need clearer state persistence, not more spectacle.
