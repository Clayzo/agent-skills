# Writing a custom shader

**You can write your own effects.** `custom-sksl` runs a fragment shader you
wrote over any node, with your own animatable parameters. It is a first-class
part of the engine, not an escape hatch, and it is where the distinctive work
lives.

It is how you get materials the built-ins have no name for — caustics, electric
arcs, flow fields, marbling, iridescence, halftone that follows the form,
dissolve edges that eat a shape from its own contours. The entire material for
a creature can be one shader over a plain circle: silhouette, shading and
motion all from sampling the input at a displaced coordinate. That is usually
less work than assembling the same look out of nodes, and it animates from a
single parameter.

Worth knowing what is already tuned before you rebuild it: `blur` into a
threshold is a metaball weld, `color-matrix` does channel thresholds and
duotones, and `dithering`/`duotone`/`pixelation` cover a lot of texture.
`effect-parameters.md` has them all. That is a reason not to rewrite a blur, not
a reason to avoid writing a shader.

## What you cannot guess

Six facts about how the engine calls your shader. Everything else is yours.

- **`main`'s coordinate is device pixels**, origin at the top left of the pass.
  Not composition units, not normalised.
- **`eval()` returns premultiplied colour.** Unpremultiply before touching
  `rgb`, or every partially transparent pixel is wrong. `unpremul()` is
  provided and uses Skia's epsilon.
- **One child, `inputImage`** — the node named by `inputId`, already drawn.
  There is no backdrop child, so a custom shader transforms its own input and
  cannot refract what is behind it. That is what `glass` is for.
- **No implicit uniforms.** Built-ins get `time` and `deviceScale` for free;
  yours gets exactly what you declared. For motion, keyframe a `phase` number —
  running it 0 → 2π across the composition also closes the loop exactly.
- **Uniforms bind positionally**, flattened in the order `parameters` declares
  them: `number` and `boolean` one float, `point` two, `color` four. The shader
  and that array have to agree.
- **Point parameters are converted to device pixels for you; numbers are not**,
  because the engine cannot tell a distance from a count. A number meaning
  "pixels" therefore stays in composition units while your coordinate is in
  device pixels, and a shader mixing the two is correct at exactly one size —
  which is how an effect drifts off its own artwork in a preview.

  **Ask for the scale.** Declare two more floats than your parameters supply,
  conventionally `uniform float2 renderScale;` **last**, and the engine appends
  the device scale. Divide by it to work in composition units:

  ```glsl
  float d = length(p - centre) / renderScale.x;
  ```

- **Your pass covers the whole canvas** unless you say otherwise, because the
  engine cannot know how far your shader reaches. `boundsPadding` (or
  `boundsPaddingX`/`boundsPaddingY`) narrows it to the input's bounds plus that
  margin. They are read by name but they are still ordinary parameters, so each
  one also takes a positional float and your shader must declare a uniform for
  it.

```glsl
uniform shader inputImage;
uniform float phase;          // keyframed 0 → 6.2832
uniform half4 tint;

half4 main(float2 p) {
  half4 src = unpremul(inputImage.eval(p));
  if (src.a < 0.001) { return half4(0.0); }
  half3 lit = /* whatever you want, in terms of p and phase */ src.rgb;
  return half4(saturate(lit) * src.a, src.a);   // premultiplied out
}
```

## One structural habit

Displace the **sample point**, not the node. Sampling `inputImage` at an offset
coordinate moves the silhouette and its shading together; translating the input
node slides the shape out from under its own material. For the same reason,
measure your shading from the displaced point rather than from screen space —
then the rim stays on the edge wherever the edge went.

That is a fact about how effects compose, so it is here. What the material
should look like is not, and that part is yours.

## Cost

Every pixel of the input runs the whole shader, and CanvasKit has no fast CPU
path for SkSL — a moderately complex shader is seconds per frame in a terminal
render and milliseconds on a GPU. Author and export from `clayzo preview`; see
`previewing-work.md`. Measure one frame before tuning a look.
