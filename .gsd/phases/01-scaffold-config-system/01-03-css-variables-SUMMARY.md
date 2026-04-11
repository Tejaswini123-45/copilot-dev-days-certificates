---
phase: "01"
plan: "03"
subsystem: "styles"
tags: [css-variables, spa-views, spinner, error-ui]
dependency_graph:
  requires: ["01-02"]
  provides: ["css-variable-system", "view-layout-classes", "loading-spinner", "error-styles"]
  affects: ["assets/app.js", "assets/styles.css"]
tech_stack:
  added: []
  patterns: ["CSS custom properties via JS setProperty", "pure-CSS keyframe animation", "SPA hidden/visible pattern"]
key_files:
  modified:
    - assets/app.js
    - assets/styles.css
decisions:
  - "CSS variable names use --primary-color convention (not --cert-* TRD prefix) per CONTEXT.md"
  - "Image URL variables wrapped in url(...) to satisfy CSS background-image format"
  - "Image variables only set when config field is truthy — prevents url(undefined) glitch"
  - "Error styles use hardcoded system-ui/neutral colors — must render even when config fails"
  - ".hidden uses !important to reliably override .view and other layout classes"
metrics:
  duration: "~5 minutes"
  completed_date: "2026-04-11"
  tasks_completed: 2
  files_modified: 2
---

# Phase 01 Plan 03: CSS Variables Summary

Dynamic CSS variable injection and base styles for the SPA layout system.

## One-liner

`applyConfigVars()` pushes all brand tokens onto `:root` via `setProperty()`; `styles.css` gains `:root` defaults, `.view`/`.hidden` layout classes, a pure-CSS spinner, and neutral error styles.

## Tasks Completed

| # | Task | Commit |
|---|------|--------|
| 1 | Add `applyConfigVars()` to `assets/app.js` | `cfdbd0a` |
| 2 | Add `:root`, view system, spinner, error styles to `assets/styles.css` | `1e09732` |

## What Was Built

### Task 1 — `applyConfigVars()` (assets/app.js)

Appended after `validateConfig()`. Sets 9 CSS custom properties on `document.documentElement.style`:

- **Colors (5):** `--primary-color`, `--secondary-color`, `--background-color`, `--text-color`, `--muted-color`
- **Border (2):** `--border-color`, `--border-width`
- **Fonts (2):** `--font-heading`, `--font-body`
- **Image URLs (3, conditional):** `--logo-url`, `--seal-url`, `--signature-url` — only set when config field is non-empty; wrapped in `url(...)` for CSS background-image compatibility

### Task 2 — CSS Blocks (assets/styles.css)

Appended after the existing reset block:

- `:root` — fallback values for all 9 variables, matching `certificate.config.json` defaults
- `body` — applies `--font-body`, `--text-color`, `--background-color`
- `.view` / `.hidden` — SPA view visibility system; `.hidden` uses `!important`
- `.loading-spinner` — 48×48px spinning ring via `@keyframes spin`, uses `var(--primary-color, #1a2e4a)` fallback
- `.error-container` / `.error-icon` / `.error-message` — neutral grey palette, `system-ui` font, no CSS variables

## Deviations from Plan

None — plan executed exactly as written.

## Self-Check: PASSED

- [x] `assets/app.js` contains `applyConfigVars()` after `validateConfig()`
- [x] `assets/styles.css` contains `:root`, `.view`, `.hidden`, `.loading-spinner`, `.error-container`
- [x] `cfdbd0a` commit exists
- [x] `1e09732` commit exists
