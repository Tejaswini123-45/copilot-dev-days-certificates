---
phase: 01-scaffold-config-system
plan: "05"
type: execute
wave: 2
depends_on:
  - "01-01"
files_modified:
  - data/sample.json
autonomous: true

must_haves:
  truths:
    - "data/sample.json is valid JSON"
    - "The filename is 'sample.json' (all lowercase)"
    - "certificate_id value follows email-sanitization convention: dots become hyphens, @ becomes -at-"
    - "All required attendee fields are present: certificate_id, name, email, workshop, date, date_iso, description"
    - "date_iso follows ISO 8601 format (YYYY-MM-DD)"
    - "certificate_id exactly matches what sanitizeId() would produce from the email field"
  artifacts:
    - path: "data/sample.json"
      provides: "Sample attendee data demonstrating the email-to-ID naming convention"
      contains: "certificate_id"
  key_links:
    - from: "data/sample.json certificate_id"
      to: "data/sample.json email"
      via: "sanitizeId(email) === certificate_id"
      pattern: "jane-doe-at-example-com"
---

<objective>
Create `data/sample.json` — the first real attendee data file. Demonstrates the email-sanitization naming convention that all attendee data files must follow.

Purpose: Satisfies Phase 01 success criteria #4 (valid JSON, email-sanitized filename convention). Provides a working test fixture for Phase 02 certificate rendering.
Output: `data/sample.json` (one attendee record with all required fields)
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
  <name>Task 1: Create data/sample.json with all required attendee fields</name>
  <files>data/sample.json</files>
  <action>
Create the directory `data/` and write `data/sample.json` with the following content:

```json
{
  "certificate_id": "jane-doe-at-example-com",
  "name": "Jane Doe",
  "email": "jane.doe@example.com",
  "workshop": "Introduction to Machine Learning",
  "date": "April 5, 2026",
  "date_iso": "2026-04-05",
  "description": "Completed 16 hours of hands-on training covering supervised learning, neural networks, and model evaluation."
}
```

**Email sanitization verification** — the `certificate_id` value MUST be what `sanitizeId()` produces from the `email` field. Walk through the algorithm manually:

1. Start: `jane.doe@example.com`
2. `.toLowerCase().trim()` → `jane.doe@example.com`
3. `.replace(/\+/g, '-plus-')` → `jane.doe@example.com` (no `+` in this email)
4. `.replace(/@/g, '-at-')` → `jane.doe-at-example.com`
5. `.replace(/\./g, '-')` → `jane-doe-at-example-com`
6. `.replace(/[^a-z0-9\-]/g, '')` → `jane-doe-at-example-com`

Result: `jane-doe-at-example-com` ✓ — this matches the `certificate_id` value above.

**Field requirements:**
- `certificate_id` — string, result of running `sanitizeId(email)`; this is also the filename key (the file that would hold this attendee's data would be named `jane-doe-at-example-com.json` in production; `sample.json` is an exception for the sample)
- `name` — full display name as it appears on the certificate
- `email` — raw email used to register; must produce `certificate_id` when sanitized
- `workshop` — full workshop name as it appears on the certificate
- `date` — human-readable date string (format: "Month D, YYYY") — displayed on certificate
- `date_iso` — machine-readable ISO 8601 date (format: "YYYY-MM-DD") — used in `<time datetime>` attribute by Phase 02
- `description` — brief completion description; displayed on certificate when `show_description: true` in config

**Critical rules:**
- Filename is `sample.json` (all lowercase — GitHub Pages Linux filesystem is case-sensitive)
- The `data/` directory is at the workspace root: `./data/sample.json`, not `./assets/data/sample.json`
- This file tests the naming convention — the filename `sample.json` is a convention exception; real files are named `{certificate_id}.json` (e.g., `jane-doe-at-example-com.json`)
  </action>
  <verify>
Run in PowerShell:
```powershell
Get-Content data/sample.json | ConvertFrom-Json | Select-Object certificate_id, name, email, workshop, date, date_iso, description
```
All 7 fields should be populated. Confirm `certificate_id` equals `"jane-doe-at-example-com"`. Confirm `date_iso` matches ISO 8601 format `YYYY-MM-DD`.
  </verify>
  <done>
`data/sample.json` exists and is valid JSON. Contains all 7 required fields. `certificate_id` value (`jane-doe-at-example-com`) is the correct sanitized form of the `email` field (`jane.doe@example.com`). `date_iso` is in `YYYY-MM-DD` format.
  </done>
</task>

</tasks>

<verification>
Run: `Get-Content data/sample.json | ConvertFrom-Json` — should parse without errors. Confirm all 7 fields are present. In browser DevTools console (with local server running), confirm:
```javascript
sanitizeId('jane.doe@example.com') === 'jane-doe-at-example-com'  // → true
```
This verifies the certificate_id value in the JSON matches the sanitize algorithm output.
</verification>

<success_criteria>
- `data/sample.json` exists at workspace root (not inside `assets/`)
- Valid JSON with all 7 fields: certificate_id, name, email, workshop, date, date_iso, description
- `certificate_id` is `"jane-doe-at-example-com"` — the correct sanitized form of `"jane.doe@example.com"`
- `date_iso` is `"2026-04-05"` (ISO 8601 format)
- Phase 01 success criteria #4 satisfied: valid JSON, filename follows email-sanitized convention
</success_criteria>

<output>
After completion, create `.gsd/phases/01-scaffold-config-system/01-05-data-convention-SUMMARY.md`
</output>
