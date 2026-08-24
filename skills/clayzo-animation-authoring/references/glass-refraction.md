# Glass and refraction

The most expensive effect in the engine and the easiest to make look like a grey
rectangle. Most of that is structural rather than a matter of taste, so the
structure comes first.

## Build it in four nodes

Glass is never one node. The shape that works, back to front:

1. **Something with structure behind it.** Not optional — see below.
2. **A mask** — a `rect`, `ellipse` or `path` filled **solid white, alpha 1**.
   This is the silhouette of the pane, not its appearance.
3. **The `glass` effect**, with `inputId` pointing at that mask.
4. **A stroked copy of the mask**, drawn on top, for the rim.

```ts
rootNodeIds: ["photo", "card-glass", "card-edge"]
//             ^ refracted   ^ effect      ^ rim on top
// "card-surface" — the mask — is NOT listed. It is reachable only as inputId.
```

Three rules hide in that list, and each produces a valid document when broken:

- **The mask must not appear in `rootNodeIds`.** An effect's `inputId` node is
  drawn *through* the effect; listing it as well draws it twice, once as a plain
  white shape sitting on top of your artwork.
- **The glass node must come after whatever it refracts.** The backdrop is a
  snapshot of the surface as already drawn, so anything later in `rootNodeIds`
  is not there yet. Put the glass first and it refracts an empty canvas.
- **The mask's fill alpha is a hard multiply on the whole result.** The shader
  ends in `half4(colored * mask.a, mask.a)`. A 40%-alpha mask does not make
  "more see-through glass" — it makes the glass itself ghost out. Translucency
  belongs in `tint.a`, and the mask stays at 1.

## It needs something to refract

Refraction offsets what is behind it. Over a flat colour, every offset samples
the same colour, fresnel and specular collapse toward zero, and only the tint
survives — a flat grey slab. This is the single most common way glass renders
"wrong" while being configured correctly.

What counts as structure: a photograph, a gradient with real range across the
pane, other artwork, a blurred colour field. What does not: a solid fill, or a
gradient so wide it is locally flat under the pane.

If the pane travels across empty margins, put something there. `clayzo-labs`
puts a photograph behind its card; `interactive-gooey-tiles` puts its lens over
a grid of photos.

## Parameters

Names and clamps are in `effect-parameters.md` — read it, because a misspelled
parameter silently does nothing. Sensible starting ranges:

| | |
|---|---|
| `blurRadius` | 3–24 px. Frosting, not focus |
| `tint` alpha | 0.06–0.28. This is where translucency lives |
| `refractionStrength` | 1–16 px. Thickness, not noise |
| `chromatic` | 0–0.06. Above that it reads as a defect |
| `fresnel` | 0.1–0.8 |
| `roughness` | 0.04 for polished, 0.2–0.35 for frosted |
| `ior` | 1.45–1.55 for anything glasslike |
| `specular` | 0.4–0.9 |

Two real configurations, both from shipped documents:

```ts
// A polished card over a photograph (clayzo-labs)
blurRadius 10 · tint rgba(0.90, 0.97, 0.98, 0.14) · refractionStrength 14
refraction true · chromatic 0.02 · fresnel 0.42 · roughness 0.04
ior 1.54 · thickness 18 · specular 0.55

// A frosted lens tracking a cursor over photos (interactive-gooey-tiles)
blurRadius 10 · tint rgba(1.00, 0.72, 0.42, 0.10) · refractionStrength 9
refraction true · roughness 0.18 · ior 1.52 · thickness 18 · specular 0.85
```

Turn `refraction` off and you have frosted glass — blur and tint, no
displacement. That is a legitimate material and it is much cheaper.

## Give it an edge

Glass with no rim reads as a blur, not an object. Draw the same path again on
top of the effect with a thin stroke — 1–1.5 px at 30–45% white — and no fill.
`clayzo-labs` does this for both the card and every shard.

## What it costs

The pass is bounded to the mask's device bounds, so cost scales with the area
the pane covers, not the canvas. Nine overlapping shards on a 1920×1080 frame
come to about 1.4× the canvas in effect pixels.

That matters most for export: SkSL runtime effects have no fast CPU path, so a
glass-heavy frame that draws in ~36 ms on a GPU takes tens of seconds in a
terminal export. Preview and export from `clayzo preview` for
anything with glass in it — see `previewing-work.md`.

## Anti-patterns

- Glass over a flat backdrop. Covered above; it is the big one.
- Blurring the glass *input*. The mask is a silhouette; blurring it softens the
  pane's own edge into nothing. Blur belongs to `blurRadius`, or to the content
  behind.
- A translucent mask fill instead of `tint.a`.
- Chromatic above ~0.06, which stops reading as glass and starts reading as a
  broken display.
- An opaque tint. At `tint.a` near 1 nothing behind the pane survives, and the
  effect is an expensive way to draw a coloured rectangle.
- Text under glass without checking contrast. Blur plus tint eats legibility;
  put text *over* the glass, as `clayzo-labs` does with its card copy.

## Interactive glass

A lens that follows the cursor is two bindings on the mask's position with a
`damp` step — a pointer wired straight to a position reads as a bug. Scale the
mask from `pointer-state.inside` through a `spring` so it opens on entry and
closes on exit; `interactive-gooey-tiles` does exactly this. See
`pointer-interaction.md`.
