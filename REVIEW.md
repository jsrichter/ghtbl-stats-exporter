# Task 6 Documentation Review

## Conclusion

Approved. The documentation is comprehensive, matches the owner interview and
current implementation, and clearly separates verified behavior from known
limitations and roadmap work.

## Blocking Issues

None.

## Verification Evidence

- `git diff --check` passes.
- The corrected home/away fallback wording now matches `index.html:684-695`:
  `README.md:67-71` and `METHODOLOGY.md:11` explain that the app assigns the
  first two IDs returned by JavaScript object-key enumeration and that numeric
  keys are normally enumerated in ascending order rather than preserved PHP
  serialization order. The ambiguity and lack of a warning are explicit.
- All relative Markdown targets referenced by the documentation exist:
  `PLAN.md`, `DATA_DICTIONARY.md`, `METHODOLOGY.md`, and
  `SPORTSPRESS_RECOMMENDATIONS.md`.
- The README quick start accurately captures the owner-provided workflow:
  WordPress **Tools > Export > All content**, hosted-app upload, optional
  filtering, and flat CSV export (`README.md:11-22`). It is usable without
  requiring source-code knowledge.
- Compatibility claims are appropriately limited to GHTBL data and Chrome
  (`README.md:9`, `README.md:65-66`), and cross-league support is presented as
  an intention rather than a verified guarantee.
- Privacy language distinguishes local in-memory processing from CDN requests,
  warns that WXR/output can contain sensitive player and site data, and tells
  contributors not to attach real exports (`README.md:55-61`,
  `README.md:89`).
- Current behavior is separated from roadmap work. The README explicitly
  identifies inferred game type, missing-player fallback without a warning,
  the ten-inning linescore limit, the 50-game box-score-sheet limit,
  nine-inning rates, and ungraceful SheetJS failure (`README.md:63-73`).
- Formula and qualification documentation matches the implementation:
  `METHODOLOGY.md:27-40` records the current total-base, AVG, OBP, SLG, OPS,
  ERA, WHIP, and K/9 formulas and the 5-game/15-AB and 5-appearance thresholds.
  It clearly labels these calculations exploratory and unsuitable as universal
  league rules.
- The data dictionary's eight fixed fields match `FLAT_STATS_HEADERS` and
  `buildFlatStatsData` in `index.html:240` and `index.html:1235-1248`.
  Derived appearance fields, preferred/dynamic field ordering, fractional-IP
  conversion, malformed-number blanking, and XLSX raw-value behavior also match
  the reviewed code paths.
- The 50-game bulk limit and note-sheet behavior described in the README and
  methodology match `index.html:1295-1316`.
- The recommendations document is constructive, scoped to portability, and
  covers the owner-requested ThemeBoy/SportsPress recommendations without
  presenting proposed functionality as current app behavior.
- There is no repository test, lint, or Markdown-validation setup. Review used
  source-to-document comparison, relative-link target checks, `git diff
  --check`, and a Node key-enumeration probe instead of an automated test suite.

## Optional Follow-ups

- `METHODOLOGY.md:7` could say that unsupported serialized values may resolve to
  `null` while parse failures resolve to an empty object. The current wording
  groups both as an empty object, but its user-facing consequence—that data may
  be omitted without a warning—is accurate.
- Once a `LICENSE` file is added, simplify the README license statement from
  intended MIT licensing to an affirmative license declaration.
