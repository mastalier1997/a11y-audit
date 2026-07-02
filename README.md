# a11y-audit

A [Claude Code](https://claude.com/claude-code) skill that audits web features
and components for accessibility against **WCAG 2.2, Level AA**.

The skill works from two complementary angles and reconciles them:

1. **Source scan** — reads component/markup/style files (JSX/TSX/Vue/Svelte/
   HTML/CSS) to catch issues visible in code: missing alt text, unlabeled
   inputs, non-semantic elements, missing `lang`, positive `tabindex`, click
   handlers on non-interactive elements, and more.
2. **Live browser** — drives the running app via
   [Playwright MCP](https://github.com/microsoft/playwright-mcp) to catch
   issues only observable at runtime: focus order, keyboard traps, visible
   focus indicators, computed contrast, dynamic ARIA state.

It writes a findings report to `a11y-report.md` — each finding tagged with the
WCAG criterion, severity, and evidence — and ends with an overall
AA-conformance verdict. It proposes concrete fixes but **always asks before
editing any file**.

## Installation

### As a plugin (recommended — works in any project)

This repo is a Claude Code plugin marketplace. In Claude Code, run:

```
/plugin marketplace add mastalier1997/a11y-audit
/plugin install a11y-audit@a11y-audit
```

The skill is then available in every session as `/a11y-audit:a11y-audit`, and
Claude also invokes it automatically when you ask about accessibility.

### As a project skill

Copy `skills/a11y-audit/SKILL.md` into your project at
`.claude/skills/a11y-audit/SKILL.md`. It's then available as `/a11y-audit` in
that project. (In this repo itself, `.claude/skills/a11y-audit` is a symlink to
the plugin copy, so both stay in sync.)

## Usage

Just ask Claude Code to check accessibility, e.g.:

- "Is the checkout form accessible?"
- "Audit the navbar component for a11y"
- "Check keyboard navigation and contrast on the settings page"

Or invoke it explicitly with a scope:

```
/a11y-audit:a11y-audit the login flow
```

If no scope is given, it audits the files touched by recent changes
(`git diff`) and the pages that render them.

### Requirements

- For the **live-browser half** of the audit, the
  [Playwright MCP server](https://github.com/microsoft/playwright-mcp) must be
  configured (e.g. in your project's `.mcp.json`) and the app must be running.
  Without it, the skill still performs the source scan and marks those
  findings as "needs runtime confirmation".

## What it checks

Organized by the four WCAG principles, applying Level AA thresholds:

- **Perceivable** — text alternatives, contrast (4.5:1 / 3:1), use of color,
  reflow at 400% zoom, text spacing, media alternatives.
- **Operable** — full keyboard operability, no keyboard traps, visible and
  unobscured focus, 24×24px target sizes, heading structure, link purpose,
  drag alternatives.
- **Understandable** — page language, associated labels, error identification
  and suggestions, consistent navigation, redundant entry, accessible
  authentication.
- **Robust** — semantic HTML, valid ARIA kept in sync at runtime, status
  messages via live regions.

This includes the success criteria new in WCAG 2.2: Focus Not Obscured
(2.4.11), Dragging Movements (2.5.7), Target Size (2.5.8), Redundant Entry
(3.3.7), and Accessible Authentication (3.3.8).

## Repository layout

```
.claude-plugin/
  plugin.json        # plugin manifest
  marketplace.json   # marketplace catalog (this repo is its own marketplace)
skills/
  a11y-audit/
    SKILL.md         # the skill definition (single source of truth)
.claude/
  skills/
    a11y-audit -> ../../skills/a11y-audit   # project-skill symlink
```
