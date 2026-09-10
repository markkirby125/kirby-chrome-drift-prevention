# kirby-chrome-drift-prevention

This skill prevents "chrome drift" (inconsistent shared navigation, headers, footers) on multi-page static sites by enforcing partials-as-source-of-truth and deterministic sync scripts.

> ### 🪄 The Magic Prompt
> Copy and paste this directly to your AI (Cursor, Windsurf, Claude Code):
> 
> ```markdown
> @agent Please install the kirby-chrome-drift-prevention skill into this workspace.
> 1. Read the `SKILL.md` file (and `references/` directory if applicable) from this repository: https://github.com/markkirby125/kirby-chrome-drift-prevention
> 2. Identify the correct rules system for our current environment (e.g., `.cursor/rules/` for Cursor, `.windsurfrules` for Windsurf, `.clinerules` for Cline, or `~/.agents/skills/` for Antigravity).
> 3. Save the contents appropriately. If our environment supports multi-file dispatcher skills, clone the directory structure exactly.
> 4. Confirm when the installation is complete.
> ```

## Manual Installation

- **Cursor:** Copy `SKILL.md` to `.cursor/rules/kirby-chrome-drift-prevention.mdc`
- **Windsurf:** Append the contents of `SKILL.md` to `.windsurfrules`
- **Antigravity:** Clone this repository to `~/.agents/skills/kirby-chrome-drift-prevention`
