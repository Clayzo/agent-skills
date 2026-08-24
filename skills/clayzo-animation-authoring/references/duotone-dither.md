# Duotone and dither

## Decision principles
- Preserve luminance hierarchy before reducing color; choose shadow/highlight colors with intentional contrast.
- Dither after tonal mapping. Use ordered patterns when deterministic pixels and stable animation matter.
- Animate threshold or palette sparingly to avoid full-frame flashing.

## Diagnostics to inspect
- Luminance/contrast range, large-flash intervals, alpha-edge leakage, exact-frame pixel determinism, and effect compiler diagnostics.

## Anti-patterns
- Quantizing before remapping, two colors with equal luminance, dither scale larger than details, or animated noise that crawls unpredictably.

## Parameter ranges
- Threshold `0.35–0.65`, contrast `0.8–2.2`, levels `3–8`, amount `0.08–0.45`, Bayer scale `1–3 px`.

## Adaptation
Choose palette from the message and background, then adjust threshold against actual coverage. Reduce amount for text and thin strokes; increase levels for gradients that must remain legible.
