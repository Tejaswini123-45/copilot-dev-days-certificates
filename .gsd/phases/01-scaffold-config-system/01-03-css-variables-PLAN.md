---
phase: 01-scaffold-config-system
plan: "03"
type: execute
wave: 3
depends_on:
  - "01-02"
files_modified:
  - assets/app.js
  - assets/styles.css
autonomous: true

must_haves:
  truths:
    - "applyConfigVars() calls document.documentElement.style.setProperty for every CSS variable — colors, border, fonts, and image URLs"
    - "Image URL variables (--logo-url, --seal-url, --signature-url) are set as 'url(...)' strings, not raw paths"
    - "applyConfigVars() only sets image variables when the config field is non-empty (conditional check)"
    - "CSS variable names match CONTEXT.md spec exactly: --primary-color, --secondary-color, --border-color, --border-width, --font-heading, --font-body"
    - ":root block in styles.css provides fallback values for all CSS variables matching config defaults"
    - ".view CSS makes each view display:block (or flex); .hidden sets display:none !important"
    - "Loading spinner uses pure CSS keyframe animation — no JS, no library dependency"
    - "Error container uses neutral grey/system colors only — no CSS variables for branding (brand config may have failed)"
  artifacts:
    - path: "assets/app.js"
      provides: "applyConfigVars() function appended after validateConfig"
      contains: "setProperty('--primary-color'"
    - path: "assets/styles.css"
      provides: ":root CSS variables, .view/.hidden classes, .loading-spinner, .error-container styles"
      contains: "--primary-color"
  key_links:
    - from: "applyConfigVars() in app.js"
      to: ":root CSS variables in styles.css"
      via: "document.documentElement.style.setProperty()"
      pattern: "setProperty\\('--primary-color'"
    - from: ".loading-spinner in index.html"
      to: "@keyframes spin animation in styles.css"
      via: "CSS class"
      pattern: "@keyframes spin"
---

<objective>
Implement dynamic CSS variable injection: `applyConfigVars()` reads the loaded config and pushes all colors, fonts, border settings, and image URLs onto `:root` via `document.documentElement.style.setProperty()`. Also adds the full view/state CSS to `styles.css`.

Purpose: After this plan, changing any visual value in `config.json` and refreshing will immediately update the rendered appearance — no HTML or CSS edits needed.
Output: `assets/app.js` (+ applyConfigVars), `assets/styles.css` (+ :root, .view/.hidden, spinner, error styles)
</objective>

<execution_context>
@.github/skills/execute-plan/SKILL.md
@.gsd/templates/summary.md
</execution_context>

<context>
@.gsd/PROJECT.md
@.gsd/ROADMAP.md
@.gsd/STATE.md
@.gsd/phases/01-scaffold-config-system/01-CONTEXT.md
@.gsd/phases/01-scaffold-config-system/01-RESEARCH.md
@.gsd/phases/01-scaffold-config-system/01-02-config-schema-SUMMARY.md
</context>

<tasks>

<task type="auto">
  <name>Task 1: Add applyConfigVars() to assets/app.js</name>
  <files>assets/app.js</files>
  <action>
Append `applyConfigVars()` to `assets/app.js` after the existing `validateConfig()` function. Do NOT remove or modify any existing content.

```javascript
// === CSS Variable Injection ===

function applyConfigVars(config) {
  const r = document.documentElement.style;

  // Colors
  r.setProperty('--primary-color',    config.primary_color    || '#1a2e4a');
  r.setProperty('--secondary-color',  config.secondary_color  || '#c8a951');
  r.setProperty('--background-color', config.background_color || '#ffffff');
  r.setProperty('--text-color',       config.text_color       || '#333333');
  r.setProperty('--muted-color',      config.muted_color      || '#777777');

  // Border
  r.setProperty('--border-color', config.border_color || '#c8a951');
  r.setProperty('--border-width', config.border_width || '7px');

  // Fonts
  r.setProperty('--font-heading', config.font_heading || "'Playfair Display', Georgia, serif");
  r.setProperty('--font-body',    config.font_body    || "'Lato', 'Helvetica Neue', sans-serif");

  // Image URLs — only set when the config field is non-empty
  if (config.logo_url)      r.setProperty('--logo-url',      `url(${config.logo_url})`);
  if (config.seal_url)      r.setProperty('--seal-url',      `url(${config.seal_url})`);
  if (config.signature_url) r.setProperty('--signature-url', `url(${config.signature_url})`);
}
```

**Rules:**
- Image URL variables MUST be wrapped in `url(...)` — CSS `background-image` expects this format
- Image variables are ONLY set when the field is truthy — prevents `url(undefined)` which causes visual glitches
- Do not set image variables with a fallback default string — missing images should simply leave the variable unset
- CSS variable names must match CONTEXT.md spec exactly (not TRD `--cert-*` prefix):
  - ✅ `--primary-color` / ❌ `--cert-primary`
  - ✅ `--border-color` / ❌ `--cert-border`
  - ✅ `--secondary-color` / ❌ `--cert-accent`
- `applyConfigVars` is called once at app boot; it is NOT re-called on SPA view transitions
  </action>
  <verify>
In browser DevTools console (with `index.html` served), run:
```javascript
fetchConfig().then(c => { applyConfigVars(c); return getComputedStyle(document.documentElement).getPropertyValue('--primary-color'); }).then(console.log);
```
Should log `#1a2e4a` (or the config value). Also verify:
```javascript
getComputedStyle(document.documentElement).getPropertyValue('--logo-url')
```
Should return `url(assets/images/logo.png)`.
  </verify>
  <done>
`applyConfigVars()` is appended to `assets/app.js`. Calling it with a valid config object sets `--primary-color` and all other CSS variables on `:root`. Image variables are wrapped in `url(...)`. Variables missing from config use fallback values.
  </done>
</task>

<task type="auto">
  <name>Task 2: Add :root variables, view layout, spinner, and error styles to assets/styles.css</name>
  <files>assets/styles.css</files>
  <action>
Append the following CSS blocks to `assets/styles.css` after the existing reset. Do NOT remove the existing reset block.

```css
/* === CSS Custom Properties === */
:root {
  --primary-color:    #1a2e4a;
  --secondary-color:  #c8a951;
  --background-color: #ffffff;
  --text-color:       #333333;
  --muted-color:      #777777;
  --border-color:     #c8a951;
  --border-width:     7px;
  --font-heading:     'Playfair Display', Georgia, serif;
  --font-body:        'Lato', 'Helvetica Neue', sans-serif;
}

/* === Body Base === */
body {
  font-family: var(--font-body);
  color: var(--text-color);
  background-color: var(--background-color);
}

/* === SPA View System === */
.view {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  min-height: 100vh;
  width: 100%;
}

.hidden {
  display: none !important;
}

/* === Loading Spinner (pure CSS — no library) === */
.loading-spinner {
  width: 48px;
  height: 48px;
  border: 5px solid #e0e0e0;
  border-top-color: var(--primary-color, #1a2e4a);
  border-radius: 50%;
  animation: spin 0.8s linear infinite;
}

@keyframes spin {
  to { transform: rotate(360deg); }
}

/* === Error View (neutral colors only — brand config may have failed) === */
.error-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 16px;
  padding: 48px 32px;
  text-align: center;
  color: #555555;
}

.error-icon {
  font-size: 48px;
  line-height: 1;
}

.error-message {
  font-size: 18px;
  font-family: system-ui, -apple-system, sans-serif;
  color: #444444;
  margin: 0;
  max-width: 400px;
}
```

**Key rules:**
- `:root` fallback values MUST match the config defaults in `certificate.config.json` exactly so the design is consistent before JS runs
- `.hidden` uses `!important` because it must override any specificity from `.view` or other layout classes
- `.loading-spinner` uses `border-top-color: var(--primary-color, ...)` — the fallback value ensures the spinner renders before JS injects config colors
- `.error-container` and `.error-message` use hardcoded `system-ui` fonts and neutral greys — NOT CSS variables — because they must render correctly even when config loading has completely failed
- Do NOT add styles for the search view or certificate view in this plan — those come in Phase 02 and 04
  </action>
  <verify>
Serve `index.html` locally. The loading spinner should be visible and animating (spinning). Inspect with DevTools: `getComputedStyle(document.querySelector('.loading-spinner')).animationName` should return `"spin"`. Inspect `:root` computed styles: `--primary-color` should be `#1a2e4a` (the CSS fallback). After calling `applyConfigVars(config)` in console, `--primary-color` on `:root` should reflect config value.
  </verify>
  <done>
`assets/styles.css` has `:root` variable declarations, `body` base styles, `.view`/`.hidden` classes, a spinning `.loading-spinner`, and neutral `.error-container` / `.error-message` styles. Spinner animates on load.
  </done>
</task>

</tasks>

<verification>
Serve locally. Confirm spinner is visible and animating. Open DevTools Computed tab on `<html>` element — `--primary-color` should show `#1a2e4a`. In console: call `fetchConfig().then(applyConfigVars)` — then change DevTools Computed view and `--primary-color` should update to the config value. Temporarily edit `config.json` primary_color to `#ff0000` and refresh — spinner color should change to red.
</verification>

<success_criteria>
- `applyConfigVars()` is in `assets/app.js`, sets all 9+ CSS variables on `:root`
- Image variables wrapped in `url(...)`, only set when config field is non-empty
- CSS variable names use `--primary-color` naming (not `--cert-*` prefix)
- `:root` in `styles.css` declares all variables with fallback values matching config defaults
- `.view` and `.hidden` classes correctly control visibility
- Loading spinner uses pure CSS `@keyframes` animation
- Error container uses system colors only (no CSS variables)
</success_criteria>

<output>
After completion, create `.gsd/phases/01-scaffold-config-system/01-03-css-variables-SUMMARY.md`
</output>
