# Pointer interaction

Authoring documents that respond to a cursor: inputs, hit areas, and the
operator graphs that connect them to properties.

## Decision principles
- Reach for a macro before writing a graph. `cursor-follow`, `look-at`,
  `hover-lift`, `press-squash`, `magnetic`, `pointer-field`, `parallax-layers`,
  and `color-on-hover` cover the shapes people ask for, already tuned.
- Blend `add` or `multiply`, not `set`, whenever the property is also animated.
  `set` silently discards the timeline for that property; the two should
  compose, so an idle loop keeps playing under a hover response.
- Smooth every pointer-driven value. A property wired straight to the cursor
  reads as a bug; a spring or a damp reads as attention. Springs for anything
  that should overshoot and settle, damping for anything that should not
  wobble — parallax and focus tracking want `damp`.
- Bound the response. Map an offset through `map-range` with clamping rather
  than feeding raw distance or a raw `angle` into a rotation, or the artwork
  flips when the cursor passes behind it.
- Give a pointer that leaves somewhere to go. Set `pointer.restOnLeave` so
  bindings relax to the authored pose instead of freezing mid-reaction, unless
  the artwork is full-bleed and the pointer should keep driving it
  (`captureOutside`).

## Structure
- Inputs are named channels the host can set (`number`, `boolean`, `trigger`,
  `point`, `color`); bindings read them by name through an `input` step.
  Declare one for anything a consumer might want to tune, and give it a range.
- Hit areas track a node's drawn bounds, so they follow whatever the artwork is
  doing rather than needing their own geometry. Use `circle` for round targets;
  `padding` grows a small target to a comfortable one.
- A binding is a list of steps. Each step chains from the previous one by
  default; name `inputs` explicitly when a step needs several operands or an
  earlier value. Operands must be declared earlier in the same list.
- Targets are a node property (with `axis` for vectors and sizes) or an effect
  parameter by id. Effect points are in composition units, like every other
  point parameter.

## Anti-patterns
- Wiring `angle` straight to a rotation with no clamp.
- Binding a `size` on a node inside a row, column, or grid: bindings apply
  after layout has run, so it moves without reflowing its siblings. Validation
  warns; restructure instead.
- Hit areas whose response grows the node they measure — the area grows too and
  the edge flickers. Keep the reaction on the node and the area on a stable
  parent, or accept that the area is measured pre-reaction (which it is).
- Springs stiff enough to ring. One rebound reads as life; three read as broken.
- Reacting to `pointer-state.down` when the intent is "pressed this element" —
  that fires anywhere on the canvas. Use `pointer-over` with `field: "pressed"`.

## Parameter ranges
- Hover scale 1.02–1.08; press scale 0.92–0.97; hover lift 4–10 units.
- Hover springs around `stiffness: 300, damping: 22`; press springs stiffer
  (450–500) so the press answers immediately.
- Follow and look-at springs around `stiffness: 120, damping: 16` — slower, so
  the motion trails the cursor rather than sticking to it.
- Damping half-lives: 0.04–0.08s for focus tracking, 0.10–0.16s for parallax.
- Head and body turns 8–16°; pupil and detail travel 8–15 units.

## Export
Bindings have no timeline representation. Video and Lottie exports bake the
document at its resting input values, so an interactive document must still
read correctly with the pointer away — compose the interaction on top of a pose
that stands on its own, rather than relying on the cursor to complete it.

## Adaptation
Decide what the pointer means before wiring anything: attention (follow,
look-at), affordance (hover, press), or material (a per-pixel field). Attention
wants slow springs and small travel; affordance wants fast, small, and
unmistakable; material wants the region to open and close rather than snap.
