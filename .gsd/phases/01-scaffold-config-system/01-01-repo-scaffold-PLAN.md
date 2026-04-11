---
phase: 01-scaffold-config-system
plan: "01"
type: execute
wave: 1
depends_on: []
files_modified:
  - index.html
  - assets/app.js
  - assets/styles.css
  - .nojekyll
  - assets/images/.gitkeep
  - assets/fonts/.gitkeep
autonomous: true

must_haves:
  truths:
    - "#loading-view has NO 'hidden' class in HTML — it is visible by default on page load"
    - "#search-view, #certificate-view, and #error-view all start with class='view hidden'"
    - "#error-view contains static message 'Could not load configuration. Please try again.' — hardcoded in HTML (not filled by JS), because branding config may have failed"
    - "#loading-view contains a child .loading-spinner div"
    - "Google Fonts preconnect and CSS link tags are present in <head>"
    - "CDN script tags for qrcode.js and html2pdf.js have defer, crossorigin='anonymous', and integrity SRI hashes"
    - "assets/app.js is linked with <script src='assets/app.js' defer> before </body> — no inline scripts anywhere"
    - ".nojekyll exists at workspace root"
  artifacts:
    - path: "index.html"
      provides: "App shell with 4 view divs, CDN links, no hardcoded text"
      contains: "loading-view"
    - path: "assets/app.js"
      provides: "JS entry point with stub init()"
      min_lines: 5
    - path: "assets/styles.css"
      provides: "CSS reset base — variables and component styles added in plan 01-03"
    - path: ".nojekyll"
      provides: "Prevents Jekyll processing on GitHub Pages"
    - path: "assets/images/.gitkeep"
      provides: "Tracks assets/images/ directory in git"
    - path: "assets/fonts/.gitkeep"
      provides: "Tracks assets/fonts/ directory in git"
  key_links:
    - from: "index.html"
      to: "assets/app.js"
      via: "<script src='assets/app.js' defer>"
      pattern: "script.*assets/app\\.js"
    - from: "index.html"
      to: "assets/styles.css"
      via: "<link rel='stylesheet' href='assets/styles.css'>"
      pattern: "link.*assets/styles\\.css"
---

<objective>
Create the repository skeleton: the HTML shell with all CDN dependencies, 4 view divs with correct initial visibility, directory structure, and base JS/CSS stubs.

Purpose: Every subsequent plan in Phase 01 assumes these files exist. This is the foundation everything else builds on.
Output: `index.html`, `assets/app.js`, `assets/styles.css`, `.nojekyll`, `assets/images/.gitkeep`, `assets/fonts/.gitkeep`
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
</context>

<tasks>

<task type="auto">
  <name>Task 1: Create index.html app shell</name>
  <files>index.html</files>
  <action>
Create `index.html` at the workspace root. All filenames are lowercase (GitHub Pages is case-sensitive), no inline scripts, no hardcoded text outside the error view.

**`<head>` contents (in order):**
1. `<meta charset="UTF-8">`
2. `<meta name="viewport" content="width=device-width, initial-scale=1.0">`
3. `<title>Workshop Certificate</title>` — JS updates this in plan 01-04 from `config.site_title`
4. `<link rel="stylesheet" href="assets/styles.css">`
5. Google Fonts (3 tags):
   ```html
   <link rel="preconnect" href="https://fonts.googleapis.com">
   <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
   <link href="https://fonts.googleapis.com/css2?family=Playfair+Display:ital,wght@0,400;0,700;1,400&family=Lato:wght@300;400;700&display=swap" rel="stylesheet">
   ```
6. CDN scripts — **fetch SRI hash from cdnjs API before writing**:
   - qrcode.js: `curl "https://api.cdnjs.com/libraries/qrcodejs/1.0.0?fields=sri"` → look for `qrcode.min.js` key → use as `integrity="sha384-..."` value
   - html2pdf.js: `curl "https://api.cdnjs.com/libraries/html2pdf.js/0.10.1?fields=sri"` → look for `html2pdf.bundle.min.js` key

   ```html
   <script src="https://cdnjs.cloudflare.com/ajax/libs/qrcodejs/1.0.0/qrcode.min.js"
           integrity="[SRI_HASH_FROM_API]"
           crossorigin="anonymous"
           defer></script>
   <script src="https://cdnjs.cloudflare.com/ajax/libs/html2pdf.js/0.10.1/html2pdf.bundle.min.js"
           integrity="[SRI_HASH_FROM_API]"
           crossorigin="anonymous"
           defer></script>
   ```

**`<body>` contents — 4 view divs:**

```html
<body>
  <!-- LOADING: no hidden class — visible by default while app initialises -->
  <div id="loading-view" class="view">
    <div class="loading-spinner" role="status" aria-label="Loading"></div>
  </div>

  <!-- SEARCH: empty — filled by JS in Phase 04 -->
  <div id="search-view" class="view hidden">
  </div>

  <!-- CERTIFICATE: empty — filled by JS in Phase 02 -->
  <div id="certificate-view" class="view hidden">
  </div>

  <!-- ERROR: static message — hardcoded because branding config may be the thing that failed -->
  <div id="error-view" class="view hidden" role="alert">
    <div class="error-container">
      <span class="error-icon" aria-hidden="true">⚠</span>
      <p class="error-message">Could not load configuration. Please try again.</p>
    </div>
  </div>

  <script src="assets/app.js" defer></script>
</body>
```

**CRITICAL RULES — do not violate:**
- `#loading-view` MUST have exactly `class="view"` — no `hidden` class
- `#search-view` and `#certificate-view` MUST be completely empty — no text content whatsoever
- Error message text IS the exception — it is deliberately hardcoded because config may have failed
- `<script src="assets/app.js" defer>` is the ONLY script tag pointing to local files
- Use relative href/src paths — no leading slashes (GitHub Pages project repos break with `/`)
  </action>
  <verify>
Open `index.html` via `npx serve .` or VS Code Live Server. The loading spinner element should be in the DOM. Browser DevTools console should show zero errors. Inspect: `document.getElementById('loading-view').classList` should NOT contain `hidden`. Inspect: `document.getElementById('search-view').innerHTML.trim()` should be `""`.
  </verify>
  <done>
`index.html` opens without console errors. Four view divs with correct IDs are present. `#loading-view` has no `hidden` class. Search and certificate views are empty. Error view has static message. All CDN links are in `<head>`.
  </done>
</task>

<task type="auto">
  <name>Task 2: Create JS stub, CSS reset, and directory placeholders</name>
  <files>assets/app.js, assets/styles.css, .nojekyll, assets/images/.gitkeep, assets/fonts/.gitkeep</files>
  <action>
Create each of the following files:

**`assets/app.js`** — minimal stub; all logic is added by plans 01-02, 01-03, 01-04:
```javascript
'use strict';

document.addEventListener('DOMContentLoaded', init);

async function init() {
  // Bootstrap sequence — implemented in plan 01-04
}
```

**`assets/styles.css`** — CSS reset only; `:root` variables and component styles are added in plan 01-03:
```css
/* === Reset === */
*, *::before, *::after {
  box-sizing: border-box;
}

html, body {
  margin: 0;
  padding: 0;
  height: 100%;
}

img {
  max-width: 100%;
  display: block;
}
```

**`.nojekyll`** — empty file at workspace root. Prevents GitHub Pages from running Jekyll, which speeds up deployment and avoids issues with directories starting with underscores.

**`assets/images/.gitkeep`** — empty file. Tracks the `assets/images/` directory in git (git does not track empty directories).

**`assets/fonts/.gitkeep`** — empty file. Tracks the `assets/fonts/` directory in git.

Final directory structure after this task:
```
/
├── index.html
├── .nojekyll
├── assets/
│   ├── app.js
│   ├── styles.css
│   ├── images/
│   │   └── .gitkeep
│   └── fonts/
│       └── .gitkeep
```
(`config/` is created by plan 01-02; `data/` is created by plan 01-05.)
  </action>
  <verify>
Run in PowerShell:
```powershell
@('assets/app.js','assets/styles.css','.nojekyll','assets/images/.gitkeep','assets/fonts/.gitkeep') | ForEach-Object { Test-Path $_ }
```
All 5 should return `True`. Check `assets/app.js` contains `'use strict'`.
  </verify>
  <done>
All 5 supporting files exist. `assets/app.js` has the `'use strict'` directive, the DOMContentLoaded listener, and a stub `init()`. `assets/styles.css` has the CSS reset. `.nojekyll` and both `.gitkeep` files exist.
  </done>
</task>

</tasks>

<verification>
Open `index.html` via a local HTTP server. Confirm: spinner is visible, no JS errors in console. In DevTools, verify `#loading-view` has class `view` only (not `view hidden`). Check source: no hardcoded text in `#search-view` or `#certificate-view`. Confirm `.nojekyll` exists at root via `Test-Path .nojekyll`.
</verification>

<success_criteria>
- `index.html` opens without JS errors
- Four view divs present: `#loading-view` (no hidden), `#search-view hidden`, `#certificate-view hidden`, `#error-view hidden`
- Error view has static "Could not load configuration" message in HTML
- Google Fonts, qrcode.js (with SRI), html2pdf.js (with SRI) loaded via CDN
- `assets/app.js` and `assets/styles.css` linked correctly
- `.nojekyll` at root, `.gitkeep` files in `assets/images/` and `assets/fonts/`
</success_criteria>

<output>
After completion, create `.gsd/phases/01-scaffold-config-system/01-01-repo-scaffold-SUMMARY.md`
</output>
