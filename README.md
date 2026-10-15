# Clayzo agent skills

Install the two public skills with the open agent skills installer:

```bash
npx skills add clayzo/agent-skills
```

The installer detects supported agents and prompts for project or global scope. To target an agent or run non-interactively, use the standard `--agent`, `--global`, and `--yes` flags.

- `clayzo-animation-authoring` routes authoring work to typed mechanics, motion, visual craft, effects, preview, and review references.
- `clayzo-animation-shipping` routes integration and export work to shipping, preflight, review, and loop references.

The top-level router files stay concise. Detailed references load only when the request needs them.
