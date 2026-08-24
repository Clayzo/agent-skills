# 2.5D orbit and camera

## Decision principles
- Use depth to clarify hierarchy and parallax while retaining 2D shape editability.
- Establish a stable camera target and depth order before animating orbit motion.
- Keep near/far planes comfortably outside content and use perspective only when it adds readable scale change.

## Diagnostics to inspect
- Camera extrema, depth-order-change samples, projected clipping, edge contact, scale discontinuity, and loop seam pairs.

## Anti-patterns
- Crossing the camera plane, frequent depth-order swaps, extreme focal lengths, unmotivated camera roll, or treating 2.5D as a mesh engine.

## Parameter ranges
- Focal length roughly `2–6×` viewport depth scale, parallax depth `5–30%` of orbit radius, camera roll usually under `5°`.

## Adaptation
Reduce perspective for dense product UI and increase it for illustrative scenes. Place orbit items explicitly, inspect every depth crossing, and preserve stable IDs for deterministic sorting.
