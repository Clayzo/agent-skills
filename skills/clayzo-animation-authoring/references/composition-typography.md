# Composition and typography

## Decision principles
- Establish focal hierarchy, reading order, safe margins, and alignment before adding motion.
- Animate text boxes as geometry; preserve line measure and baseline relationships.
- Use contrast and negative space to separate roles instead of relying on effects.

## Diagnostics to inspect
- Text-box clipping, alpha bounds, edge clipping, coverage, luminance range, tree depth, and visible timing. No OCR inference is available.

## Anti-patterns
- Centering every element, long line lengths, text touching frame edges, animated tracking that harms legibility, or motion that reverses reading order.

## Parameter ranges
- Body line length `45–75` characters, safe margin `4–8%` of short edge, line height `1.2–1.6×` font size, display tracking changes under `0.08em`.

## Adaptation
Respond to actual copy length, language, aspect ratio, and platform. Inspect declared text boxes on all review backgrounds and revise hierarchy rather than shrinking everything uniformly.
