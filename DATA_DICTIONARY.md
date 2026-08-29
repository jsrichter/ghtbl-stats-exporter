# Flat Export Data Dictionary

Flat CSV and XLSX outputs contain one row for each nonzero player ID found in each team's `sp_players` data for a filtered game. The first eight columns are fixed; statistic columns vary with the source file.

## Fixed metadata columns

| Column | Meaning | Source or derivation |
| --- | --- | --- |
| `gameID` | WordPress event post ID | `wp:post_id` on `sp_event` |
| `season` | Event season | First `sp_season` category |
| `date` | Event date (`YYYY-MM-DD`) | Date portion of `wp:post_date` |
| `team name` | Team assigned to this row | Referenced `sp_team` title; may fall back to an ID or `?` |
| `team side` | `away` or `home` | Inferred as described in [METHODOLOGY.md](METHODOLOGY.md) |
| `player id` | WordPress player post ID | Key in the team's serialized `sp_players` data |
| `player name` | Player display name | Matching `sp_player` title, otherwise `Player #<id>` |
| `game type` | `regular` or `playoff` | Keyword inference described in [METHODOLOGY.md](METHODOLOGY.md) |

## Derived appearance columns

| Column | Values | Meaning |
| --- | --- | --- |
| `batting_g` | `0` or `1` | A recognized batting field exists, or `status=lineup` without a pitching field |
| `pitching_g` | `0` or `1` | At least one recognized pitching field exists |

These indicate detected appearances in one event row, not a SportsPress career or season games-played total.

## Common batting columns

The exporter prioritizes `h`, `ab`, `r`, `rbi`, `bb`, `so`, `lob`, `doubles`, `triples`, `hr`, `sb`, `hbp`, `sac`, and `sf`. Values come from event player-stat metadata, subject to CSV cleanup below.

## Common pitching columns

| Field | Typical meaning |
| --- | --- |
| `pitcher_ip` | Innings pitched in baseball notation (`2.1` means two innings and one out) |
| `pitcher_h` | Hits allowed |
| `pitcher_r` | Runs allowed |
| `pitcher_er` | Earned runs |
| `pitcher_bb` | Walks allowed |
| `pitcher_so` | Strikeouts |
| `pitcher_hr` | Home runs allowed |
| `pitcher_hbp` | Hit batters |
| `pitcher_gs` | Games started indicator |
| `pitcher_w` | Win indicator |
| `pitcher_l` | Loss indicator |
| `pitcher_sv` | Save indicator |
| `pitcher_cg` | Complete game indicator |
| `pitcher_sho` | Shutout indicator |

Exact source-field semantics depend on the SportsPress Baseball configuration.

## Other dynamic columns

Every additional player-stat key is included after preferred fields, sorted alphabetically. This commonly includes `status` and `sub`. Column sets can differ between exports or filter selections, so consumers should bind fields by header name rather than position. The internal `__order` marker is never exported.

## CSV-specific cleanup

For recognized numeric fields (`batting_g`, `pitching_g`, common batting fields, and common pitching fields), CSV:

- retains valid signed integer and decimal tokens;
- converts `pitcher_ip` values `1/3`, `2/3`, `N 1/3`, and `N 2/3` to `0.1`, `0.2`, `N.1`, and `N.2`;
- blanks malformed nonnumeric values; and
- prefixes text beginning with `=`, `+`, `-`, or `@` with an apostrophe to reduce formula injection risk.

XLSX preserves source values and does not apply this CSV-only cleanup. A blank CSV numeric cell can mean either a blank source value or malformed input that was discarded.
