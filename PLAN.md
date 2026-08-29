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

### 1C. Identify each exported team as home or away

Status: complete

Goal: Add one explicit flat-export column identifying whether each player-stat
row belongs to the home team or the away team.

Files:

- `index.html`

Implementation:

- Carry the side (`away` or `home`) from the existing game-team iteration into
  every row emitted by `buildFlatStats`.
- Add a `team side` metadata column to `buildFlatStatsData`, adjacent to `team
  name`, so both flat CSV and XLSX exports receive the same layout.
- Update CSV numeric-stat column indexing and XLSX column widths for the added
  metadata column.
- Keep the change localized to flat-stat exports; do not alter team parsing,
  game classification, box-score exports, dashboards, or filters.

Acceptance checks:

- Every flat CSV and XLSX player row has `away` for the game’s away team and
  `home` for the game’s home team.
- The CSV header and every data row have the same number of columns.
- Numeric stat validation, valid negative values, malformed numeric blanking,
  and text-field formula hardening still apply to the correct columns.
- Existing `game type` values and dynamic statistic ordering remain unchanged.
- `git diff --check` passes.

Implementation notes:

- Added `away`/`home` side labels to the existing flat-stat team iteration and
  carried the label into each emitted player row.
- Inserted `team side` immediately after `team name` in the shared CSV/XLSX
  flat-data layout, added its XLSX width, and shifted CSV numeric-stat lookup
  from seven to eight metadata columns.
- Source-level acceptance checks passed for both side mappings, metadata and
  row placement, the eight-column CSV numeric offset, unchanged numeric
  validation and formula hardening, and continued use of `gameSeasonType`.
  The production diff remains localized to flat-export construction.
- `git diff --check` passes. A temporary browser assertion harness was removed
  after headless Chrome could not complete reliably in the execution
  environment; no test-only code remains.

### 1B. Identify regular-season and playoff games in flat exports

Status: complete

Goal: Add an explicit game-type column to each exported flat-stat row so users
can distinguish regular-season records from playoff records.

Files:

- `index.html`

Implementation:

- Reuse the existing `gameSeasonType(game)` classifier so exported values match
  the regular/playoff filter already shown in the UI.
- Carry the classified value into every player row emitted by `buildFlatStats`.
- Add a `game type` metadata column to `buildFlatStatsData`, before the dynamic
  statistic columns, with values `regular` or `playoff`.
- Keep CSV numeric-field handling aligned with the new metadata-column count so
  stat validation and formula hardening continue to apply to the right columns.
- Keep the change localized to flat CSV/XLSX data generation; do not alter XML
  parsing, classification rules, box-score exports, dashboards, or filters.

Acceptance checks:

- Every flat CSV data row has a `game type` value.
- A known Playoff Tournament event exports `playoff` and a known regular-season
  event exports `regular`.
- The CSV header and every data row have the same number of columns.
- Valid negative numeric stats remain numeric-looking, malformed numeric stats
  remain blank, and text-field formula hardening still works after the offset
  change.
- The flat XLSX export uses the same metadata layout and remains valid.
- `git diff --check` passes.

Implementation notes:

- Added `gameSeasonType(g)` to every player row built by `buildFlatStats` and
  exposed it as `game type`, the seventh flat-export metadata column, in both
  CSV and XLSX data.
- Shifted CSV numeric-stat detection from six to seven metadata columns and
  added a matching XLSX column width, leaving stat ordering and all non-flat
  export paths unchanged.
- A headless-browser check against the application functions passed regular and
  playoff classification, equal header/data column counts, valid negative and
  malformed numeric handling, text formula hardening, XLSX metadata placement,
  and XLSX workbook serialization.
- The supplied export contains representative player-stat events 26926
  (`Jets @ Mets`, Regular Season) and 27693 (`Playoff Game 1`, Playoff
  Tournament), which exercise the classifier's `regular` and `playoff` paths.
- `git diff --check` passes.

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

### 6. Publish comprehensive user and contributor documentation

Status: complete

Goal: Make the project understandable and usable by GHTBL administrators,
other SportsPress Baseball leagues, analysts, and prospective contributors.

Files:

- `README.md`
- New `DATA_DICTIONARY.md`
- New `METHODOLOGY.md`
- New `SPORTSPRESS_RECOMMENDATIONS.md`

Implementation:

- Rewrite the README around the primary workflow: export all WordPress content,
  open the hosted or local app, upload the XML, filter the results, and export a
  flat CSV for Access, Python, R, or another analysis tool.
- Explain that the app is a lightweight, client-side interpreter for WordPress
  WXR exports containing SportsPress and SportsPress Baseball data.
- Document the live site, local and GitHub Pages use, supported input records,
  views, filters, export types, known limitations, privacy model, CDN/network
  requirements, Chrome-only testing status, troubleshooting, contributing, and
  GitHub Issues support.
- Describe aggregate statistics and leaderboards as exploratory current behavior,
  not authoritative league results; do not imply that fixed nine-inning pitching
  rates are correct for every league.
- Add a data dictionary for flat-export metadata, derived appearance fields, and
  dynamically discovered SportsPress statistic columns.
- Add a methodology document covering parsing, team-side and playoff inference,
  appearance detection, numeric cleanup, formulas, qualification thresholds, and
  other transformations or assumptions.
- Add a constructive recommendations document for ThemeBoy/SportsPress describing
  export shortcomings and improvements that would make baseball data portable.
- State the intended MIT license, credit `jsrichter`, invite pull requests and
  cross-league compatibility reports, and ask users to provide descriptions rather
  than attaching XML fixtures containing league data.
- Include a concise roadmap summary in the README and link to this file for detail.

Acceptance checks:

- A nontechnical league administrator can follow the documented export/upload/CSV
  workflow without reading source code.
- Current behavior is separated clearly from roadmap work and known limitations.
- The README does not claim broad browser or cross-league compatibility that has
  not been tested.
- Privacy wording says XML is processed in memory and discarded on refresh/close,
  while also disclosing requests to public CDN providers and the sensitivity of
  downloaded player data.
- Documentation does not promise a missing-record warning that is not implemented.
- All internal links resolve and `git diff --check` passes.

Implementation notes:

- Rewrote the README around the all-content export, hosted/local upload,
  filtering, and flat CSV workflow, with privacy, CDN, compatibility,
  troubleshooting, contribution, license-intent, and roadmap guidance.
- Added a flat-export data dictionary, implementation-based methodology, and
  constructive SportsPress portability recommendations.
- Separated current behavior from roadmap work, including inferred sides and
  game type, missing-player fallback without a warning, nine-inning exploratory
  rates, ten-inning parsing, and the 50-game box-sheet limit.

### 7. Improve portability, validation, and league-specific behavior

Status: proposed

Goal: Remove assumptions that are reliable for GHTBL but may be incorrect for
other SportsPress Baseball leagues.

Files:

- `index.html`
- `README.md`
- `METHODOLOGY.md`

Implementation:

- Validate home/away identification and report when title-based matching fails.
- Show visible warnings for referenced player or team records missing from the
  WordPress export.
- Rework the 50-game per-sheet bulk-export limit and support games longer than 10
  innings.
- Remove or redesign calculated summary rates and leaderboards so the app does not
  silently impose nine-inning formulas or GHTBL qualification rules on other
  leagues.
- Test parsing and exports against current SportsPress Baseball installations from
  leagues other than GHTBL, using locally sanitized reproduction data only.

Acceptance checks:

- Users can distinguish exact source values from inferred or calculated values.
- League-specific calculation rules are configurable, explicitly selected, or not
  calculated.
- Ambiguous and incomplete source records produce actionable feedback.
- Long games and large filtered exports do not silently omit expected output.

## Coder Handoff

Implement Task 1C only.

- Read `.agents/coder.md` and this file before editing.
- Keep the patch localized to flat export row/data construction, XLSX column
  widths, and the CSV numeric-column offset.
- Use exactly one new `team side` column with `away`/`home` values.
- Exercise every acceptance scenario in Task 1C.
- Mark Task 1C complete here only after those checks pass.
- Record assumptions and checks directly under Task 1C.

## Checker Handoff

Review the coder's Task 1C changes only.

- Read `.agents/checker.md`, this file, and the final diff.
- Verify every Task 1C acceptance check using representative away and home rows.
- Confirm the inserted metadata column does not shift CSV numeric validation,
  weaken text formula hardening, or disturb the existing game-type column.
- Confirm unrelated display, filtering, parsing, and box-score behavior is
  unchanged.
- Write findings and test evidence to `REVIEW.md`; distinguish blocking findings
  from optional follow-ups.
