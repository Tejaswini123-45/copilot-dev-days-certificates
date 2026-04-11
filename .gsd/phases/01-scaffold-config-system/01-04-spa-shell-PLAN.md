---
phase: 01-scaffold-config-system
plan: "04"
type: execute
wave: 4
depends_on:
  - "01-03"
files_modified:
  - assets/app.js
  - index.html
autonomous: true

must_haves:
  truths:
    - "Opening index.html with no query param: loading spinner shows briefly, then transitions to search-view"
    - "Opening index.html with ?id=any-value: loading spinner shows briefly, then transitions to certificate-view"
    - "Config load failure (unreachable or invalid JSON): loading spinner shows briefly, then error-view is shown — no uncaught JS exception in console"
    - "sanitizeId('jane.doe@example.com') returns 'jane-doe-at-example-com'"
    - "showView('error-view') hides all other views and shows only error-view"
    - "document.title is updated from config.site_title after successful config load"
    - "init() calls showView('loading-view') as its very first synchronous statement"
    - "All user-visible text values set via .textContent — never .innerHTML"
  artifacts:
    - path: "assets/app.js"
      provides: "Complete SPA bootstrap: init(), showView(), getQueryParam(), sanitizeId()"
      exports: ["init", "showView", "getQueryParam", "sanitizeId"]
  key_links:
    - from: "init() first statement"
      to: "showView('loading-view')"
      via: "synchronous call before any await"
      pattern: "showView\\('loading-view'\\)"
    - from: "init() catch block"
      to: "showView('error-view')"
      via: "catch (err) handler"
      pattern: "showView\\('error-view'\\)"
    - from: "document.addEventListener('DOMContentLoaded')"
      to: "init()"
      via: "listener callback"
      pattern: "DOMContentLoaded.*init"
---

<objective>
Complete the SPA bootstrap: implement `showView()`, `getQueryParam()`, `sanitizeId()`, and the full `init()` function. After this plan, opening `index.html` produces a complete view transition cycle — loading → correct view (or error on failure).

Purpose: This wires all the pieces from plans 01-01 through 01-03 together. The app should now satisfy Phase 01 success criteria #1 and #5, and lay the groundwork for Phase 02 and 04 rendering.
Output: `assets/app.js` (complete), `index.html` (no changes to structure, only verified correct)
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
@.gsd/phases/01-scaffold-config-system/01-03-css-variables-SUMMARY.md
</context>

<tasks>

<task type="auto">
  <name>Task 1: Add showView(), getQueryParam(), and sanitizeId() to assets/app.js</name>
  <files>assets/app.js</files>
  <action>
Append the three utility functions to `assets/app.js` after the existing `applyConfigVars()` function. Do NOT modify existing content.

```javascript
// === SPA View System ===

function showView(activeId) {
  const viewIds = ['loading-view', 'search-view', 'certificate-view', 'error-view'];
  viewIds.forEach(function (id) {
    const el = document.getElementById(id);
    if (el) el.classList.toggle('hidden', id !== activeId);
  });
}

// === URL Utilities ===

function getQueryParam(name) {
  const params = new URLSearchParams(window.location.search);
  return params.get(name);
}

function sanitizeId(email) {
  return email
    .toLowerCase()
    .trim()
    .replace(/\+/g, '-plus-')
    .replace(/@/g, '-at-')
    .replace(/\./g, '-')
    .replace(/[^a-z0-9\-]/g, '');
}
```

**Rules for `showView()`:**
- Iterates ALL 4 view IDs every call — adds `hidden` to inactive views, removes it from the active view
- Uses `classList.toggle(class, force)` — the boolean second argument is the toggle direction
- Gracefully handles missing elements (`if (el)` guard) — safe to call before DOM is fully assembled
- `#loading-view` starts without `hidden` in HTML; `showView('search-view')` will add `hidden` to it

**Rules for `sanitizeId()`:**
- Email: `jane.doe@example.com` → ID: `jane-doe-at-example-com`
- Step order matters: `+` first (`-plus-`), then `@` (`-at-`), then `.` (`-`), then strip remaining non-alphanumeric/hyphen
- `toLowerCase()` and `trim()` called before any replacements
- This function is used in Phase 04 for email input → URL navigation; it must produce the same result as the attendee JSON filename

**Rules for `getQueryParam()`:**
- Uses `URLSearchParams` (built-in, no polyfill needed for modern browsers)
- Returns `null` if the parameter is absent (native behavior)
  </action>
  <verify>
In browser DevTools console (served via local server):
```javascript
// Test sanitizeId
sanitizeId('jane.doe@example.com')  // → "jane-doe-at-example-com"
sanitizeId('user+tag@domain.co')    // → "user-plus-tag-at-domain-co"

// Test showView
showView('error-view');
// error-view should appear; loading-view, search-view, certificate-view should all have class 'hidden'
document.getElementById('loading-view').classList.contains('hidden')  // → true
document.getElementById('error-view').classList.contains('hidden')    // → false

showView('loading-view');
// loading-view should be visible again
document.getElementById('loading-view').classList.contains('hidden')  // → false
```
  </verify>
  <done>
`showView()`, `getQueryParam()`, and `sanitizeId()` are all present in `assets/app.js`. `sanitizeId('jane.doe@example.com')` returns `'jane-doe-at-example-com'`. `showView('error-view')` correctly hides all other views.
  </done>
</task>

<task type="auto">
  <name>Task 2: Replace stub init() with full bootstrap implementation</name>
  <files>assets/app.js</files>
  <action>
Replace the existing stub `init()` function in `assets/app.js` with the complete implementation below. The stub was:
```javascript
async function init() {
  // Bootstrap sequence — implemented in plan 01-04
}
```

Replace it with:

```javascript
async function init() {
  showView('loading-view');

  const rawId = getQueryParam('id');
  const id = rawId ? sanitizeId(rawId) : null;

  try {
    const config = await fetchConfig();
    validateConfig(config);
    applyConfigVars(config);

    if (config.site_title) {
      document.title = config.site_title;
    }

    if (id) {
      // Phase 02 will fetch attendee data and render the certificate view
      // fetchAttendee(id) + renderCertificateView(config, attendee) — not yet implemented
      showView('certificate-view');
    } else {
      // Phase 04 will render the search landing page
      // renderSearchView(config) — not yet implemented
      showView('search-view');
    }
  } catch (err) {
    console.error('[App] init error:', err);
    showView('error-view');
  }
}
```

**Rules:**
- `showView('loading-view')` MUST be the first statement — synchronous, before any `await`
- `validateConfig(config)` is called immediately after `await fetchConfig()` — an invalid config is treated the same as a network failure (both land in `catch`)
- `applyConfigVars(config)` is called before transitioning to any view — CSS variables should be set before content renders
- If `config.site_title` is present, update `document.title` — this satisfies "org_name in rendered UI" requirement via `site_title` which includes org name
- The `if (id)` branch is a stub — no `fetchAttendee` call — it only transitions the view. Phase 02 will fill in the rendering
- The `else` branch is also a stub — no search rendering — Phase 04 will fill it in
- `catch` block: logs error to console with `[App]` prefix for debugging, then shows `error-view`
- Do NOT use `.innerHTML` anywhere in this file — use `.textContent` for all text assignments

**IMPORTANT: `init` function order in file.** After this plan, `assets/app.js` contains these declarations in order:
1. `'use strict';`
2. `document.addEventListener('DOMContentLoaded', init);`
3. `async function init() { ... }` ← REPLACED stub
4. `// === Config Loader ===`
5. `async function fetchConfig() { ... }`
6. `function validateConfig(config) { ... }`
7. `// === CSS Variable Injection ===`
8. `function applyConfigVars(config) { ... }`
9. `// === SPA View System ===`
10. `function showView(activeId) { ... }`
11. `// === URL Utilities ===`
12. `function getQueryParam(name) { ... }`
13. `function sanitizeId(email) { ... }`

All functions are hoisted `function` declarations — calling order in `init` does not require forward-declaration concerns.
  </action>
  <verify>
Serve locally. Test these scenarios:

1. **No ?id param** → Open `http://localhost:PORT/` → Should briefly show spinner, then `#search-view` should be visible (even if empty), no console errors
2. **With ?id param** → Open `http://localhost:PORT/?id=test` → Should briefly show spinner, then `#certificate-view` should be visible (even if empty), no console errors  
3. **Config error** → Temporarily rename `config/certificate.config.json` to `config/certificate.config.json.bak` → Refresh → Spinner should show, then **error view** should appear with the message — no uncaught exception in console. Restore the config file after testing.

Check: `document.title` after load should be `"Workshop Certificates — Acme Workshop Co."` (from `config.site_title`).
  </verify>
  <done>
`init()` in `assets/app.js` is fully implemented. Opening with no query param shows search-view. Opening with `?id=x` shows certificate-view. Renaming/removing config causes error-view to show. `document.title` is updated from config. No uncaught JS exceptions in any scenario.
  </done>
</task>

</tasks>

<verification>
Full smoke test (serve locally):
1. Visit `/` → loading spinner appears, transitions to search-view (empty, no errors)
2. Visit `/?id=jane-doe-at-example-com` → loading spinner appears, transitions to certificate-view (empty, no errors)
3. Temporarily break config (rename file) → error-view shows, no uncaught exception
4. Check `document.title` = "Workshop Certificates — Acme Workshop Co."
5. In console: `sanitizeId('jane.doe@example.com') === 'jane-doe-at-example-com'` → `true`
</verification>

<success_criteria>
- `init()` is fully implemented: loading → config fetch → applyVars → correct view
- No uncaught JS exceptions in normal or error scenarios
- `showView()` correctly toggles all 4 view divs
- `sanitizeId()` produces correct email-to-ID transformation
- `document.title` updated from `config.site_title` after config loads
- Phase 01 success criteria #1 satisfied: loading indicator → correct view, no blank screen
- Phase 01 success criteria #5 satisfied: config failure shows error state, not thrown exception
</success_criteria>

<output>
After completion, create `.gsd/phases/01-scaffold-config-system/01-04-spa-shell-SUMMARY.md`
</output>
