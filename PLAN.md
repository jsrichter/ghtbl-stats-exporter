# Small-Improvements Roadmap

This roadmap keeps the current static, single-file architecture. Tasks are ordered
by user impact and are intentionally small enough to implement and review one at a
time.

## Repository Snapshot

- `index.html` contains the complete UI, XML parser, statistics logic, and export
  logic.
- `README.md` documents local use and GitHub Pages deployment.
- `GHTBL-export.xml` is an approximately 85 MB real-world input sample.
- There is no automated test, lint, formatting, or dependency-update setup.

## Tasks

### 1A. Make CSV numeric fields compatible with Microsoft Access

Status: complete

Goal: Prevent Access type-conversion failures when numeric stat columns contain
malformed source values or valid negative numbers are changed into text.

Files:

- `index.html`

Observed evidence:

- Access record numbers map to CSV data records, one line after the header.
- The failing records contain malformed numeric values including `o`, ``1` ``,
  a lone backtick, ``3` ``, `0=`, and `q`.
- A `bb` value of `-1` is exported as `'-1` by the general spreadsheet
  formula-hardening rule, converting a syntactically valid number to text.

Implementation:

- Define the known numeric stat fields from the existing batting, pitching, and
  derived games-played field lists.
- In CSV output only, preserve syntactically valid decimal and negative numeric
  values without adding a formula-hardening apostrophe.
- Continue converting valid fractional baseball innings such as `1/3`, `2/3`,
  `2 1/3`, and `2 2/3` to decimal-style innings notation used by the exporter.
- Emit an empty field when a known numeric stat contains malformed nonnumeric
  source text.
- Keep formula hardening for nonnumeric/text fields beginning with `=`, `+`,
  `-`, or `@`.
- Do not alter XLSX exports, XML parsing, dashboards, or displayed box scores.

Acceptance checks:

- The eight reported Access records no longer contain text in their affected
  numeric columns.
- `-1`, `0`, `.1`, `2.1`, and other valid numeric tokens remain numeric-looking
  CSV values without a leading apostrophe.
- Malformed tokens found in the reported rows serialize as empty CSV fields.
- Valid fractional innings serialize consistently.
- Text fields still receive spreadsheet-formula hardening.
- `git diff --check` passes.

Implementation notes:

- Added a CSV-only numeric field set from the existing batting, pitching,
  `batting_g`, and `pitching_g` fields.
- CSV numeric stats now retain trimmed, syntactically valid negative and decimal
  tokens, convert valid fractional `pitcher_ip` values, and blank malformed
  numeric text. CSV cell hardening remains enabled for formula-like text fields.
- A temporary headless-browser check passed 19 cases covering `-1`, `0`, `.1`,
  `2.1`, all four supported fractional-innings forms, the reported malformed
  tokens (`o`, `1` followed by a backtick, a lone backtick, `3` followed by a
  backtick, `0=`, and `q`), and text beginning with `=`, `+`, `-`, or `@`.
- The supplied CSV was inspected directly: affected records 10,958, 11,771,
  12,721, 17,172, 18,988, 20,231, and 22,891 map to the tested numeric fields
  and tokens (with `1` followed by a backtick appearing in two records);
  record 10,935's numeric `bb=-1` maps to the tested numeric-negative path.
  The separate text `sub=-1` value remains formula-hardened as intended.
- XLSX builders and all display paths were left unchanged. `git diff --check`
  passes.

### 1. Validate uploaded XML and report read failures

Status: complete

Goal: Keep invalid or unsupported files from silently opening an empty results
view.

Files:

- `index.html`

Implementation:

- Detect `<parsererror>` after `DOMParser.parseFromString`.
- Treat an export with no SportsPress teams, players, or events as unsupported
  input and show a useful error.
- Handle `FileReader.onerror`.
- Keep the upload control available after an error so the user can retry.
- Do not change valid-file parsing or the application architecture.

Acceptance checks:

- A malformed XML file produces a visible error and remains on the upload screen.
- A valid XML file with no supported SportsPress records produces a visible,
  specific error.
- A valid SportsPress export still reaches the game list.
- Loading state is cleared in every failure path.

Implementation notes:

- Added parser-error and empty SportsPress export validation, plus a shared
  retryable upload error state.
- Added `FileReader.onerror` handling; all parse, validation, and read failures
  clear loading before rendering the upload control and safely escaped message.
- Verified each failure branch keeps `S.step` on `upload`, and verified the
  included valid export contains 20 teams, 1,010 players, and 1,130 events for
  the unchanged valid parsing path.
- `git diff --check` passes. No JavaScript test runner is installed in this
  repository, so verification was limited to branch inspection and fixture
  record checks.

### 2. Improve basic document and control accessibility

Status: proposed

Goal: Make the existing interface easier to navigate with browsers, keyboards,
and assistive technology.

Files:

- `index.html`

Implementation:

- Add `lang`, viewport metadata, and a descriptive `<title>`.
- Associate text labels or `aria-label` values with filters and icon-only
  controls.
- Make game rows keyboard-operable, or render them as semantic buttons.
- Expose loading and error messages through an appropriate live region.
- Preserve the current visual design.

Acceptance checks:

- Every interactive control has an accessible name.
- A game can be selected without a pointing device.
- Browser zoom and narrow mobile layouts use the device viewport correctly.

### 3. Add lightweight tests for pure data helpers

Status: proposed

Goal: Protect the highest-risk stat calculations without introducing a build
framework or redesigning the app.

Files:

- `index.html`
- New minimal test file(s)
- `README.md`

Implementation:

- Add focused cases for PHP unserialization, innings-to-outs conversion, batting
  and pitching appearance detection, rate calculations, and CSV formula
  hardening.
- Prefer a dependency-free Node test runner if the installed/runtime baseline
  supports it.
- Document one command that runs the tests.

Acceptance checks:

- Tests run locally with one documented command.
- Tests cover normal, blank, and malformed inputs.
- The browser application still works when `index.html` is opened directly.

### 4. Replace the large export with a small, sanitized fixture

Status: proposed

Goal: Reduce clone size and remove ambiguity about whether a full WordPress
export contains personal or production data.

Files:

- `GHTBL-export.xml`
- New fixture file
- `.gitignore`
- `README.md`

Implementation:

- Create a minimal synthetic SportsPress export containing enough data for one
  representative game.
- Verify that it exercises teams, players, results, batting, pitching, and a
  missing-player fallback.
- Remove the large export only after the fixture is verified.
- Ignore local full-export filenames or place them under an ignored directory.

Acceptance checks:

- The synthetic fixture loads successfully in the app.
- No real names, email addresses, URLs, or other production data remain.
- Repository size is materially reduced for future clones.

Note: Removing the tracked full export is intentional and should be confirmed by
the repository owner before implementation.

### 5. Document browser and external-dependency expectations

Status: proposed

Goal: Make operational requirements and failure modes clear.

Files:

- `README.md`
- Optionally `index.html`

Implementation:

- Document supported modern-browser features, including `findLastIndex`.
- Explain that CDN access is required for styling, icons, fonts, and XLSX export.
- Document the behavior when SheetJS is unavailable.
- Consider a small runtime check that disables export buttons with a clear
  message when `XLSX` did not load.

Acceptance checks:

- Users can tell which features require network access.
- A missing XLSX dependency does not fail with an unexplained runtime exception.

## Coder Handoff

Implement Task 1A only.

- Read `.agents/coder.md` and this file before editing.
- Keep the patch localized to CSV value serialization.
- Do not modify XLSX or display behavior and do not begin other tasks.
- Exercise every acceptance scenario in Task 1A, including the reported rows.
- Mark Task 1A complete here only after those checks pass.
- Record assumptions and checks directly under Task 1A.

## Checker Handoff

Review the coder's Task 1A changes only.

- Read `.agents/checker.md`, this file, and the final diff.
- Verify all Task 1A acceptance checks against the reported CSV records.
- Confirm valid negative and decimal values remain numeric-looking.
- Confirm malformed numeric tokens become blank without weakening formula
  hardening for text fields.
- Check that XLSX and display paths are unchanged.
- Write findings and test evidence to `REVIEW.md`; distinguish blocking findings
  from optional follow-ups.
