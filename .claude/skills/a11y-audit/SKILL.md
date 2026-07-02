---
name: a11y-audit
description: >
  Audit web features and components for accessibility against WCAG 2.2 AA.
  Checks both the running app in a real browser (via Playwright MCP) and the
  source files (JSX/TSX/Vue/Svelte/HTML/CSS). Use when the user asks to check
  accessibility, a11y, screen-reader support, keyboard navigation, contrast,
  ARIA, or whether a component/feature is accessible or WCAG-compliant. Reports
  findings and proposes fixes, but asks before applying any change.
---

# Accessibility Audit (WCAG 2.2 AA)

You audit web UI for accessibility against **WCAG 2.2, Level AA** (which
includes all Level A success criteria). You work from two complementary
angles and reconcile them:

1. **Source scan** — read the component/markup/style files to catch issues
   that are visible in code (missing alt text, unlabeled inputs, non-semantic
   elements, hardcoded colors, missing lang, positive tabindex, click handlers
   on non-interactive elements).
2. **Live browser** — drive the running app via Playwright MCP to catch issues
   only observable at runtime (focus order, keyboard traps, visible focus
   indicators, computed contrast, dynamic ARIA state, motion).

Neither alone is sufficient. Source tells you intent; the browser tells you
what a real assistive-tech user actually gets. Report a finding as confirmed
when both agree; note it as "needs runtime confirmation" when only source
suggests it.

## Scope

Audit the feature or component the user names. If none is named, audit the
files touched by recent changes (`git diff`), then the page(s) that render
them. Don't try to audit the whole app in one pass — it produces a shallow
report. Go deep on the named scope.

## What to check (grouped by WCAG principle)

Apply the Level AA thresholds noted below.

**Perceivable**
- Text alternatives: every `img`, icon, and non-text control has a meaningful
  `alt`/`aria-label`; decorative images are `alt=""` / `aria-hidden`.
- Contrast (1.4.3, AA): normal text ≥ **4.5:1**, large text ≥ **3:1**. UI
  components and graphical objects (1.4.11) ≥ **3:1**. Measure computed
  foreground/background in the browser, not from the hex in source alone.
- Use of color (1.4.1): color is not the only visual means of conveying
  information (no color-only status).
- Reflow (1.4.10): content reflows and stays usable at 400% zoom / 320px width
  with no loss of content or function.
- Text spacing (1.4.12) and content on hover/focus (1.4.13) behave correctly.
- Media: captions for prerecorded audio, and audio description for prerecorded
  video, where applicable.

**Operable**
- Full keyboard operability (2.1.1): every interactive element reachable and
  usable by keyboard, in a logical order, with **no keyboard trap** (2.1.2).
- Visible focus indicator on every focusable element (2.4.7 Focus Visible), and
  focus is not obscured (2.4.11, new in 2.2).
- Target size (2.5.8, AA, new in 2.2): interactive targets are at least
  **24x24 CSS px** (or have adequate spacing); flag smaller.
- No content that flashes more than three times per second (2.3.1).
- Headings and labels are descriptive (2.4.6); heading hierarchy is well-formed
  (single h1, no skipped levels).
- Link purpose is clear from context (2.4.4).
- Dragging movements have a single-pointer alternative (2.5.7, new in 2.2).

**Understandable**
- `html[lang]` set (3.1.1); language of parts marked where it changes (3.1.2).
- Inputs have programmatically associated labels; errors are identified in text
  (3.3.1) with suggestions (3.3.3); labels/instructions provided (3.3.2).
- Consistent navigation (3.2.3) and consistent identification (3.2.4).
- Redundant entry (3.3.7) and accessible authentication (3.3.8) — both new in
  2.2 — are respected in forms and login flows.

**Robust**
- Valid, semantic HTML; ARIA used only to fill gaps, never to override native
  semantics. No invalid ARIA roles/attributes; required ARIA states present and
  kept in sync at runtime (4.1.2).
- Status messages exposed via live regions (4.1.3) — `aria-live`, roles like
  `status`/`alert` — so screen readers announce them without focus change.

## Runtime checks to actually perform in the browser

- Tab through the feature start to finish; record the focus order and whether
  it matches the visual order.
- Confirm every interactive element is reachable and activatable with keyboard
  alone (Enter/Space), and that focus never gets stuck.
- Confirm a visible focus indicator appears on each stop and isn't hidden
  behind sticky headers/overlays.
- Read the accessibility snapshot (`browser_snapshot`) to inspect the exposed
  role/name/state of each control.
- Trigger dynamic states (open a menu, submit an invalid form) and confirm ARIA
  state updates and that messages are announced via a live region.
- Spot-check computed contrast on text and UI components against the AA ratios.

## Output — findings report

Write findings to `a11y-report.md` and return a summary. For each finding:

- **Component / location**: file + line if from source, or URL + element
  (role/name) if from browser.
- **WCAG criterion**: number, name, and level (e.g. `1.4.3 Contrast (Minimum)
  — AA`).
- **Severity**: blocker / serious / moderate / minor.
- **Evidence**: the exact code snippet or the observed runtime behavior
  (measured ratio, focus-order note, missing announcement, screenshot filename
  if visual).
- **Source vs. runtime**: confirmed by both, source-only (needs runtime
  confirmation), or runtime-only.

Order findings by severity. End with a count by severity, and an overall
verdict: **AA-conformant** or **not AA-conformant** (with the blocking
criteria listed).

## Proposing fixes — ask before applying

Do **not** edit files automatically. For each fixable finding, present a
concrete proposed change:

- The specific edit (semantic element swap, `aria-label` addition, contrast
  token change with a compliant value, focus-management fix, etc.).
- What it fixes and any trade-off or visual impact.

Then ask the user which fixes to apply. Group them so they can approve in
batches (e.g. "apply all blockers", "apply the contrast fixes"). Apply only
what they approve, then offer to re-run the audit on the changed scope to
verify.

## Notes

- Automated checks catch a minority of real issues. Your keyboard and
  screen-reader-semantics walkthrough is the high-value part — don't skip it in
  favor of a quick source grep.
- Be honest about confidence: a computed contrast ratio is a hard fact; whether
  a label is "descriptive enough" is a judgment call — say which is which.
