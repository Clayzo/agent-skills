# Product microinteractions

## Decision principles
- Confirm input immediately, preserve spatial continuity, and reserve larger motion for meaningful state changes.
- Make enter, update, success, error, and exit states distinguishable without relying on motion alone.
- Keep interruption safe: new input should retarget rather than queue stale animations.

## Diagnostics to inspect
- First response tick, landing tick, contrast range, flashes, frozen intervals, clipped focus/labels, and reduced-motion alternatives.

## Anti-patterns
- Delayed acknowledgement, celebratory motion on routine actions, hover-only meaning, repeated bounce, or animated layout shift.

## Parameter ranges
- Press feedback: 60–120 ms and scale `0.96–0.99`.
- Tooltip/menu entry: 100–220 ms. Success emphasis: 250–500 ms with at most one overshoot.

## Adaptation
Tune amplitude to action frequency and risk. Frequent controls need quieter motion; destructive or asynchronous actions need clearer state persistence, not more spectacle.
