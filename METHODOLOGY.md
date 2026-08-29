# Methodology and Assumptions

This describes current implementation, not official GHTBL or SportsPress policy.

## Parsing

The browser parses WXR XML and makes three passes through `<item>` records: `sp_team` maps IDs to names; `sp_player` maps IDs to names, positions, numbers, and the `bt` metric; and `sp_event` supplies event metadata, serialized results, and player stats. The small PHP unserializer supports arrays, strings, integers, booleans, nulls, and doubles. Malformed or unsupported serialized content resolves to an empty object and can be omitted without a visible warning.

## Team-side inference

For results with at least two team IDs, the event title is authoritative when it matches `Away Team @ Home Team`. Names are normalized for punctuation/case, and suffix matching permits shortened names. If both names cannot be matched to distinct IDs, the app assigns the first two IDs returned by JavaScript object-key enumeration as home and away, respectively. The result IDs are numeric-like keys and are therefore normally enumerated in ascending numeric order, not necessarily in their original PHP-serialized order. This fallback is ambiguous and is neither validated nor reported.

## Game-type inference

Season, league, and title text are joined and lowercased. A game is `playoff` if the text contains `playoff`, `postseason`, `championship`, `final`, `semifinal`, `quarterfinal`, or `wild card`; otherwise it is `regular`. This powers the filter and flat column.

## Player rows and appearances

Every nonzero player ID with a stat object for the inferred teams becomes a row. A missing player record is labeled `Player #<id>` without a visible warning.

A pitching appearance has any recognized pitching field. A batting appearance has any recognized batting field, or `status=lineup` and no pitching field. Flat output records these as `pitching_g` and `batting_g`; aggregate games count distinct events among detected appearances.

## Innings

Baseball decimal notation is interpreted as outs: the integer part contributes three outs per inning and the decimal part at most two. Aggregated outs convert back to `innings.outs`. Linescores read fields `one` through `ten` and display at least seven innings.

## Exploratory aggregate formulas

- Total bases: `H + 2B + (2 x 3B) + (3 x HR)`
- AVG: `H / AB`
- OBP: `(H + BB + HBP) / (AB + BB + HBP + SF)`
- SLG: `total bases / AB`
- OPS: `OBP + SLG`
- ERA: `(ER x 9) / IP`
- WHIP: `(BB + H) / IP`
- K/9: `(SO x 9) / IP`

Blank or nonnumeric inputs add zero. Batting rates display three decimal places; pitching rates display two. Fixed nine-inning ERA and K/9 do not reflect GHTBL's seven-inning games and should not be treated as authoritative or universally portable.

Home run and OPS leaderboards require 5 detected batting games and 15 at-bats. ERA requires 5 pitching games. Leaderboards show 8 entries; player batting and pitching tables show up to 50 rows after sorting. These are fixed GHTBL-oriented defaults, not configurable league rules.

## Flat transformation and scope

Fixed metadata is followed by appearance flags, preferred baseball fields, and other discovered stat keys alphabetically. See [DATA_DICTIONARY.md](DATA_DICTIONARY.md). CSV validates recognized numeric fields, normalizes supported fractional innings, blanks malformed numeric tokens, and hardens formula-like text. XLSX preserves raw values.

Season, inferred type, team, and inclusive date filters apply to display and export. Bulk XLSX includes all filtered games in summary and flat sheets, but creates box-score sheets for only the first 50 games and records truncation in a note sheet.
