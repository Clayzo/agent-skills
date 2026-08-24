# Self-review

## Decision principles
- Review the exact current revision with semantic samples and preserve the rendered packet for direct visual inspection.
- Treat diagnostics as evidence: fix errors, investigate warnings, and acknowledge only intentional residual warnings with a concrete reason.
- Re-review after every edit; never carry approval across revisions.

## Diagnostics to inspect
- Blank frames, renderer errors/warnings, alpha bounds, clipping, contrast, motion energy, freezes, flashes, loop seams, render p95, and temporary surfaces.

## Anti-patterns
- Judging from one hero frame, acknowledging warning categories not present, approving stale frames, using PNG byte size as blankness, or invoking a second model as a substitute for inspection.
- Approving wallpaper-only or orb-only frames when the brief asked for a product surface, lock screen, notification, clock, or named copy.
- Treating abstract pills as a substitute for status, type, avatars, or button labels.

## Parameter ranges
- Use `8–12` semantic samples, inspect all error frames, and keep review record/token pages within declared hard budgets.

## Adaptation
Focus the review range when iterating one transition, then run the whole composition before submission. Explain why each remaining warning is acceptable for this design and revision.
