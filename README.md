# kirby-chrome-drift-prevention

*This skill is part of the [Kirby Skills Collection](https://github.com/markkirby125/kirby-skills-collection).*

An AI agent skill that prevents "chrome drift" (inconsistent shared navigation, headers, footers) on multi-page static sites. It enforces partials-as-source-of-truth and provides rules for deterministic sync scripts.

## Key Features

- **Drift Detection:** Mechanical rules for spotting inconsistencies across flat HTML files.
- **Partials-as-Source-of-Truth:** Guidelines for extracting canonical elements into single partial files.
- **Deterministic Syncing:** Requirements for building a script to inject partials cleanly with a mandatory `--check` mode.
- **Pre-Flight Verification:** A strict checklist for content preservation, block nesting, and tag balance.

## Tech Stack

- **Format**: Markdown / YAML
- **Compatibility**: Antigravity, Claude Code, Cursor, Windsurf, Cline

## Prerequisites

- An AI coding assistant or agent runner.
- A multi-page static site (flat HTML) lacking native templating or server-side rendering.

## Getting Started

### 1. Installation via The Magic Prompt

Copy and paste this directly to your AI (Cursor, Windsurf, Claude Code):

```markdown
@agent Please install the kirby-chrome-drift-prevention skill into this workspace.
1. Read the `SKILL.md` file (and `references/` directory if applicable) from this repository: https://github.com/markkirby125/kirby-chrome-drift-prevention
2. Identify the correct rules system for our current environment (e.g., `.cursor/rules/` for Cursor, `.windsurfrules` for Windsurf, `.clinerules` for Cline, or `~/.agents/skills/` for Antigravity).
3. Save the contents appropriately. If our environment supports multi-file dispatcher skills, clone the directory structure exactly.
4. Confirm when the installation is complete.
```

### 2. Manual Installation

- **Cursor:** Copy `SKILL.md` to `.cursor/rules/kirby-chrome-drift-prevention.mdc`
- **Windsurf:** Append the contents of `SKILL.md` to `.windsurfrules`
- **Antigravity:** Clone this repository to `~/.agents/skills/kirby-chrome-drift-prevention`

## Architecture

This skill is structured as a single-file Markdown document containing YAML frontmatter and the drift prevention standard operating procedure (SOP).

### Directory Structure

```
├── SKILL.md      # The main skill definition and instructions
├── README.md     # Project documentation
└── .gitignore    # Ignored files
```

### Skill Logic Flow

1. **Step 0: Census:** Counts pages carrying the chrome element inside structural containers to map variant fingerprints.
2. **Step 1: Extract Canonical Partials:** Isolates the majority variant into partial files (e.g., `partials/nav.html`).
3. **Step 2: Encode Variant Rules:** Maps page-path patterns to specific partial rules.
4. **Step 3: Write the Sync Script:** Defines strict requirements for the script (deterministic, scoped replacement, `--check` mode, idempotent).
5. **Step 4: Review Divergence:** Forces review of the `--check` report before any writes occur.
6. **Step 5: Migration Run:** Executes the write mode and validates through mechanical gates (content preservation, tag balance, idempotence).
7. **Step 6 & 7: Ongoing Protocol:** Establishes the process for future edits to prevent recurrence of drift.

## Usage

The agent triggers this skill when working on static site navigation, headers, footers, or when tasked with mass HTML edits.

To force execution, tell the agent:
> "We need to fix the navigation across our static HTML site. Follow the kirby-chrome-drift-prevention skill."

## Troubleshooting

### False Drift Reports
**Issue:** The sync script detects drift where none exists.
**Solution:** Check for exact-string anchors and whitespace differences. Re-extract the current bytes before asserting, and operate on containing elements instead of raw strings.

### Invalid HTML Post-Sync
**Issue:** Browsers render the site differently, but the files look correct.
**Solution:** Check for block elements (`<h3>`, `<ul>`) injected inside `<p>` tags. Browsers auto-close paragraphs, causing the DOM to diverge from the source. Close the paragraph before injecting blocks.

## External Resources & Authority Links
- [MDN Web Docs: Document Object Model (DOM) Architecture](https://developer.mozilla.org/en-US/docs/Web/API/Document_Object_Model)
- [W3C HTML Standard & Tag Nesting Rules](https://html.spec.whatwg.org/)
- [Jamstack Architecture Principles](https://jamstack.org/what-is-jamstack/)
- [Cloudflare Pages Documentation](https://developers.cloudflare.com/pages/)
