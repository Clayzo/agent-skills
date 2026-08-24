# Product UI craft

Build product surfaces that a fluent user trusts at first glance. Typed nodes, accurate marks, readable chrome, and restrained motion matter more than decorative atmosphere.

## Decision principles

- Match the named operating system, application, and surface. Follow its spacing, type sizes, materials, controls, and copy instead of inventing generic chrome.
- Use accurate brand marks and interface icons. Use packaged or supplied vector paths; never substitute emoji, a letter in a circle, or a vaguely similar glyph.
- Copy is geometry. Brief strings such as times, dates, names, prices, gates, and button labels remain readable text or carefully outlined paths.
- Structure drawables beneath groups or nulls and animate the parent rig. A useful hierarchy is scene → region → item controller → drawable.
- Hold, then commit. Product motion usually needs a readable still followed by one physical change, with 40–54ms sibling staggering where appropriate.
- Preserve identity through a morph or transition. Shared labels, marks, and values should feel like the same object moving, not two objects cross-fading.
- Glass samples an actual backdrop. Shadows stay close to the surface and follow the implied light and tilt.
- Start with the fewest objects. Improve scale, weight, brightness, and spacing before adding glow, cards, or background decoration.

## Diagnostics to inspect

- The first, middle, and settled frames each read as intentional product screenshots.
- Named copy is present, legible, and aligned to the surrounding controls.
- Marks remain recognizable at their final display size.
- Mixed-size rows are optically centered and text stacks use a consistent vertical step.
- Status clusters, camera cutouts, safe areas, and trailing insets do not collide.
- Stacked cards have a clear newest-to-oldest order with believable scale and overlap.

## Anti-patterns

- Wallpaper, meshes, or glowing orbs becoming the subject.
- Generic app glyphs, inaccurate status marks, or duplicated clocks.
- Flat translucent fills presented as refractive glass.
- One easing curve and duration applied to every property.
- Fading siblings when an intentional hold-cut or shared-element move is clearer.
- Flattening the rig and keying every path directly.
- Adding advanced effects before the surface itself is accurate.

## Useful ranges

- Press feedback: 80–120ms, scale `0.96–0.99`.
- Product transition: 160–280ms. Emphasis: 350–500ms.
- UI spring: stiffness 160–200, damping 24–32, at most one visible overshoot.
- Sibling stagger: 10–13 ticks at 240 ticks per second.
- Notification or card corner radius: 20–36px, adjusted for scale.
- Glass blur: 12–24px with a restrained tint and a contact shadow whose softness is roughly ten times its offset.

## Adaptation

Use the supplied product references and assets before inventing a surface. When choosing between atmosphere and accurate chrome, copy, marks, or morph topology, keep the accurate product detail.
