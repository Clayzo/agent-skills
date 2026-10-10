# Clayzo agent skills

Install the two public skills with the open agent skills installer:

```bash
npx skills add clayzo/agent-skills
```

The installer detects supported agents and prompts for project or global scope. To target an agent or run non-interactively, use the standard `--agent`, `--global`, and `--yes` flags.

- `clayzo-animation-authoring` routes authoring work to typed mechanics, motion, visual craft, effects, preview, and review references.
- `clayzo-animation-shipping` routes integration and export work to shipping, preflight, review, and loop references.

The top-level router files stay concise. Detailed references load only when the request needs them.

## Claude plugin

This repository is also the Clayzo plugin for Claude. After adding it, connect Clayzo from the plugin's **Connectors** tab and sign in with your Clayzo account. Claude can then render frames to check its work and export animations with the Clayzo CLI, including in Claude's sandbox.

The skills install the `clayzo` CLI from npm and run it where Claude works. The connector, `https://app.clayzo.com/api/clayzo/mcp`, gives Claude a render pass for your account that lasts an hour. Your animations and exported files are not sent to Clayzo.
