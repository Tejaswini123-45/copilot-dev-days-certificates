---
phase: 01-scaffold-config-system
plan: "02"
type: execute
wave: 2
depends_on:
  - "01-01"
files_modified:
  - config/certificate.config.json
  - assets/app.js
autonomous: true

must_haves:
  truths:
    - "config/certificate.config.json is valid JSON containing all 35 schema fields from the research spec"
    - "fetchConfig() fetches from 'config/certificate.config.json' — relative path, no leading slash"
    - "fetchConfig() throws an Error (not console.error) when the HTTP response is not ok"
    - "validateConfig() checks 'org_name', 'primary_color', and 'certificate_title' and throws descriptive Error on any missing field"
    - "Both functions are top-level 'function' declarations appended to assets/app.js"
  artifacts:
    - path: "config/certificate.config.json"
      provides: "Full flat config schema with org, certificate, feature, search, PDF, and SEO fields"
      contains: "org_name"
    - path: "assets/app.js"
      provides: "fetchConfig and validateConfig functions added to existing stub"
      exports: ["fetchConfig", "validateConfig"]
  key_links:
    - from: "assets/app.js fetchConfig()"
      to: "config/certificate.config.json"
      via: "fetch('config/certificate.config.json')"
      pattern: "fetch\\('config/certificate\\.config\\.json'\\)"
    - from: "assets/app.js validateConfig()"
      to: "required field list"
      via: "for-of loop over required array"
      pattern: "for.*of.*required"
---

<objective>
Define the complete configuration schema and wire the JS config loader. Creates `config/certificate.config.json` with all fields needed across all phases, and adds `fetchConfig()` and `validateConfig()` to `assets/app.js`.

Purpose: Later plans (css-variables, spa-shell) and later phases (02–06) depend on this config structure being locked. Defining it fully now prevents schema churn.
Output: `config/certificate.config.json` (complete flat schema), `assets/app.js` (+ fetchConfig, validateConfig)
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
@.gsd/phases/01-scaffold-config-system/01-01-repo-scaffold-SUMMARY.md
</context>

<tasks>

<task type="auto">
  <name>Task 1: Create config/certificate.config.json with full flat schema</name>
  <files>config/certificate.config.json</files>
  <action>
Create the directory `config/` and write `config/certificate.config.json` with the complete flat schema below. This schema covers all phases (01–06) — it is fully defined now. Do not use nested objects; all keys are at the root level.

```json
{
  "org_name": "Acme Workshop Co.",
  "org_tagline": "Learn. Build. Grow.",
  "org_website": "https://example.com",
  "logo_url": "assets/images/logo.png",

  "primary_color": "#1a2e4a",
  "secondary_color": "#c8a951",
  "background_color": "#ffffff",
  "text_color": "#333333",
  "muted_color": "#777777",
  "border_color": "#c8a951",
  "border_width": "7px",

  "seal_url": "assets/images/seal.png",
  "signature_url": "assets/images/signature.png",
  "font_heading": "'Playfair Display', Georgia, serif",
  "font_body": "'Lato', 'Helvetica Neue', sans-serif",

  "certificate_title": "Certificate of Completion",
  "pre_name_text": "This is to certify that",
  "post_name_text": "has successfully completed the workshop",
  "issued_by_label": "Authorized By",
  "date_label": "Date of Completion",
  "id_label": "Certificate ID",
  "signature_name": "Dr. Jane Smith",
  "signature_title": "Workshop Director",

  "show_qr": true,
  "show_description": true,
  "show_seal": true,

  "search_headline": "Verify Your Certificate",
  "search_subtext": "Enter the email address you used to register.",
  "search_placeholder": "your@email.com",
  "search_button": "Find My Certificate",
  "search_footer_note": "Can't find yours? Contact us at hello@example.com",

  "pdf_filename_prefix": "certificate",
  "pdf_format": "a4",
  "pdf_orientation": "landscape",
  "pdf_margin": 0,

  "site_title": "Workshop Certificates — Acme Workshop Co.",
  "og_image_url": "assets/images/og-default.png",
  "twitter_handle": "@acmeworkshops",
  "template_version": "1.0.0"
}
```

**Key rules:**
- `border_width` is a string with unit: `"7px"` not `7` — the CSS variable injection sets it directly, so the unit must be included
- `font_heading` and `font_body` are CSS font-family strings with single quotes inside double-quoted JSON
- `show_qr`, `show_description`, `show_seal` are JSON booleans (not strings)
- `pdf_margin` is a number (not string) — used by html2pdf.js margin option directly
- All image paths are relative (no leading slash): `"assets/images/logo.png"` not `"/assets/images/logo.png"`
  </action>
  <verify>
Run: `Get-Content config/certificate.config.json | ConvertFrom-Json` in PowerShell — should parse without errors and return an object. Spot-check: the result should have `org_name`, `primary_color`, `show_qr`, `pdf_margin`, `template_version` properties.
  </verify>
  <done>
`config/certificate.config.json` exists, is valid JSON, and contains all 35 fields including org, color, certificate, feature flags, search copy, PDF, and SEO fields.
  </done>
</task>

<task type="auto">
  <name>Task 2: Add fetchConfig() and validateConfig() to assets/app.js</name>
  <files>assets/app.js</files>
  <action>
Append the following two functions to `assets/app.js`, after the existing `init()` stub. Do NOT overwrite the existing content — append below it.

```javascript
// === Config Loader ===

async function fetchConfig() {
  const res = await fetch('config/certificate.config.json');
  if (!res.ok) throw new Error(`Config fetch failed: ${res.status}`);
  return res.json();
}

function validateConfig(config) {
  const required = ['org_name', 'primary_color', 'certificate_title'];
  for (const field of required) {
    if (!config[field]) throw new Error(`Invalid config: missing required field "${field}"`);
  }
}
```

**Rules:**
- Use `fetch('config/certificate.config.json')` — relative path, NO leading slash (breaks on GitHub Pages project repos under a subpath)
- `fetchConfig` throws an `Error` (not `console.error`, not silent) so the `init()` catch block can intercept it
- `validateConfig` checks for falsy values (`!config[field]`), which catches both `undefined` and empty string
- Both are top-level `function` declarations (hoisted) — not arrow functions assigned to variables
- Do not call these functions here; `init()` will call them in plan 01-04

After editing, `assets/app.js` should contain:
1. `'use strict';` at top
2. `document.addEventListener('DOMContentLoaded', init);`
3. `async function init() { /* stub */ }`
4. `// === Config Loader ===`
5. `async function fetchConfig() { ... }`
6. `function validateConfig(config) { ... }`
  </action>
  <verify>
Open Browser DevTools console on `index.html` (served locally). Run: `fetchConfig().then(c => console.log(c.org_name))` — should log `"Acme Workshop Co."`. Run `validateConfig({org_name:'x',primary_color:'y',certificate_title:'z'})` — should complete without error. Run `validateConfig({org_name:'x'})` — should throw.
  </verify>
  <done>
`assets/app.js` contains `fetchConfig()` and `validateConfig()` as top-level functions after the init stub. `fetchConfig()` correctly returns config JSON when served locally. `validateConfig()` throws on missing required fields.
  </done>
</task>

</tasks>

<verification>
Serve the project locally. Open DevTools. Run `fetchConfig().then(console.log)` — should display the full config object. Confirm `config.org_name === "Acme Workshop Co."`, `config.show_qr === true` (boolean), `config.border_width === "7px"` (string with unit), `config.pdf_margin === 0` (number). Run `validateConfig({})` — should throw an Error with "org_name" in the message.
</verification>

<success_criteria>
- `config/certificate.config.json` is valid JSON with all 35 fields
- `border_width` is a string (`"7px"`), `pdf_margin` is a number (`0`), feature flags are booleans
- `fetchConfig()` fetches from relative path `config/certificate.config.json`
- `fetchConfig()` throws on non-ok HTTP response
- `validateConfig()` throws descriptive Error when `org_name`, `primary_color`, or `certificate_title` is missing
- Both functions appended to `assets/app.js` without removing existing content
</success_criteria>

<output>
After completion, create `.gsd/phases/01-scaffold-config-system/01-02-config-schema-SUMMARY.md`
</output>
