# Motion fundamentals

## Decision principles
- Animate hierarchy and causality, not decoration. One dominant movement should explain each state change.
- Use integer ticks and deliberate landings. Prefer transform/opacity tracks over layout-changing motion.
- Match easing to intent: Bezier for directed transitions, spring for interruptible physical response, hold for explicit cuts.

## Diagnostics to inspect
- Semantic keyframe/landing samples, motion energy, frozen intervals, flashes, loop seam difference, and edge clipping.
- Track count, first/last ticks, overshoot samples, and exact end values.

## Anti-patterns
- Identical easing and duration on every element.
- Unbounded spring oscillation, accidental frozen spans, movement without visual causality, or keyframes outside visibility.

## Parameter ranges
- Product transitions: 120–320 ms. Emphasis: 300–600 ms.
- Bezier control points generally remain in `[0,1]`; spring damping ratio should avoid more than one visible rebound unless expressive.

## Adaptation
Start from content hierarchy, distance, density, and input modality. Scale duration with travel distance and simplify motion when several regions compete; never copy timing without reviewing semantic frames.
