# Recommendations for ThemeBoy and SportsPress

The standard WordPress **Tools > Export > All content** file contains SportsPress Baseball data, but administrators must reconstruct relationships across posts, taxonomies, and PHP-serialized metadata before obtaining an ordinary game/player table. These changes would make the data easier to archive, audit, migrate, and analyze.

## Provide first-party tabular export

Offer CSV and XLSX tables for teams, players, games/results, and player-game stats. Stable IDs should connect tables, with season, league, team, and date filters.

## Publish a schema and data dictionary

Document each baseball key, type, unit, null behavior, and whether it is entered, derived, or aggregated. Include schema/version metadata and migration notes. Explicitly document baseball innings notation.

## Encode semantics directly

Export explicit home and away team IDs and a stable game-type or competition-stage value, instead of requiring title or insertion-order inference. Every player-stat row should carry event, team, player, and appearance-role IDs. Report missing references during export.

## Use a portable interchange format

Avoid opaque PHP serialization for interoperability. Use documented JSON in WXR metadata or, preferably, purpose-built JSON/CSV. Represent meaningful batting and pitching order with explicit fields.

## Make calculations league-aware

Export raw counts and league rules needed to reproduce rates. Document formulas and let administrators configure the innings basis rather than assuming nine innings.

## Validate export quality

Report missing team/player records, malformed numbers, ambiguous sides, incomplete results, and unknown stat keys. A machine-readable warnings file would distinguish clean and partial exports.

## Support practical export sizes

Allow scoped exports that include dependencies for selected games. Streaming or paginated APIs and documented limits would be more reliable than loading an entire site export into browser memory.

## Publish fictional compatibility fixtures

Provide small fixtures covering regulation and extra-inning games, substitutions, multiple pitchers, playoffs, missing values, and custom keys. Third-party tools could then test safely without real player data.

These changes would reduce support burden while giving leagues durable ownership of their records and safer integration with Access, Excel, Python, R, and archival systems.
