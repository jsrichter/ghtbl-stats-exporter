# GHTBL Stats Exporter

GHTBL Stats Exporter is a lightweight, client-side interpreter that turns a WordPress WXR (`.xml`) export containing SportsPress and SportsPress Baseball records into usable baseball data. It was created for the Greater Hartford Twilight Baseball League (GHTBL), where the standard WordPress export did not provide an easy path from SportsPress data to a flat, analysis-ready file.

The primary output is a flat CSV suitable for Microsoft Access, Python, R, or other analysis tools. The app also provides Excel exports, game box scores, filters, dashboards, and exploratory statistics.

Use the hosted app: **https://jsrichter.github.io/ghtbl-stats-exporter/**

> Compatibility status: this is currently a personal/internal utility tested with GHTBL data and current software versions in Google Chrome. It is intended to work with other SportsPress Baseball sites, but cross-league and broad browser compatibility have not yet been established.

## Quick start: export a flat CSV

1. Sign in to the WordPress administration area for the league site.
2. Go to **Tools > Export**.
3. Select **All content**, then download the XML export file. SportsPress data is interdependent, so a full-site export is currently required and may be large.
4. Open the [hosted app](https://jsrichter.github.io/ghtbl-stats-exporter/) in Chrome.
5. Drop the `.xml` file onto the upload area, or click the area and select it.
6. After parsing finishes, optionally filter by season, game type, team, or date.
7. Select **Flat stats**, then **Export CSV**.
8. Import the downloaded CSV into Access, Python, R, or the tool of your choice.

The filters apply to exports as well as on-screen views. See the [data dictionary](DATA_DICTIONARY.md) before building an import schema.

## Run it locally

Download or clone the repository and open `index.html` directly in Chrome. There is no build step and no local server is required. The app can also be deployed as a static site through GitHub Pages.

Internet access is still needed for the complete experience because the page loads Tailwind CSS, Tabler Icons, Google Fonts, and SheetJS from public CDNs. Parsing and CSV export use browser features, but styling and icons may be missing offline, and XLSX exports require SheetJS to load successfully.

## Expected input

The app accepts one WordPress WXR XML file containing:

- `sp_team` records for team IDs and names
- `sp_player` records for player IDs, names, positions, numbers, and metrics
- `sp_event` records for games, results, statistics, seasons, leagues, and venues

It reads serialized metadata including `sp_results`, `sp_players`, `sp_status`, and `sp_metrics`. Malformed XML, or XML containing no supported SportsPress teams, players, or events, produces an upload error.

## Views, filters, and exports

The **Games** view shows a filtered dashboard and game list. Selecting a game opens its linescore, batting table, pitching table, and a single-game XLSX export.

The **Stats** view aggregates team, batter, and pitcher data and displays leaderboards. These figures are exploratory, not authoritative league standings or official statistics. Their formulas and qualifications are documented in [METHODOLOGY.md](METHODOLOGY.md).

The **Flat stats** view produces one row per player/game/team record:

- **CSV** is the recommended output for Access and general analysis. Known numeric stat columns are cleaned to reduce type-conversion failures.
- **XLSX** contains the same flat layout without CSV-specific numeric cleanup.
- **Export all** creates an XLSX workbook with an all-games summary, flat stats, and individual box-score sheets for up to 50 filtered games.
- A selected game can be exported as its own box-score workbook.

Available filters are season, inferred regular/playoff type, team, start date, and end date.

## Privacy

Uploaded XML is read and parsed in browser memory. This project does not upload the file to an application server, and the in-memory data is discarded when the page is refreshed or closed.

WordPress exports can contain player information and unrelated site content. Treat both source XML and downloaded output as sensitive, and do not attach real league exports to public issues or pull requests.

Opening the app also requests Tailwind CSS, Tabler Icons, Google Fonts, and SheetJS from public CDN providers. Those providers receive ordinary web request metadata, but the app does not send the uploaded XML to them.

## Current limitations

- Only GHTBL exports and Google Chrome have been tested.
- The full WordPress **All content** export is required; large-file performance depends on available browser memory and has no published supported-size limit.
- Home/away assignment first matches `Away Team @ Home Team`. If that fails,
  the app assigns the first two enumerated result IDs as home and away,
  respectively. Because numeric object keys are normally enumerated in ascending
  numeric order by JavaScript, this does not reliably preserve SportsPress's
  serialized order. No warning identifies this ambiguous fallback.
- Regular/playoff status is inferred from season, league, and title text.
- Missing player records retain their rows as `Player #<id>`, but no visible incomplete-record warning is shown.
- Linescores parse innings one through ten only.
- Bulk export includes box-score sheets for only the first 50 filtered games and adds a note when more were omitted. Summary and flat-stat sheets still cover the full filtered set.
- Aggregate rates and leaderboards impose fixed formulas and qualifications, including nine-inning ERA and K/9. They may be unsuitable for seven-inning or differently configured leagues.
- If SheetJS fails to load, XLSX actions may fail without a friendly explanation.

See [METHODOLOGY.md](METHODOLOGY.md) for exact transformations and assumptions.

## Troubleshooting

**Unsupported file:** Confirm WordPress export was set to **All content** and contains SportsPress `sp_team`, `sp_player`, or `sp_event` records.

**Invalid XML or unreadable file:** Download a fresh export and retry. Do not edit WXR in a word processor.

**No games match:** Clear the filters, especially date and game type.

**XLSX export fails:** Confirm internet access and that the SheetJS CDN is not blocked. CSV does not depend on SheetJS.

**A name is `Player #123`:** The event references a player whose `sp_player` record was absent.

For help, open a [GitHub issue](https://github.com/jsrichter/ghtbl-stats-exporter/issues) with expected and actual behavior, browser/OS, WordPress and SportsPress versions, approximate XML size, and console errors. Do **not** attach a real XML export; describe the relevant structure or provide fictional, sanitized reproduction data.

## Contributing and roadmap

Pull requests and compatibility reports from other SportsPress Baseball leagues are welcome. Priorities include sanitized test data, reliable home/away validation, missing-record warnings, configurable or removed league-specific calculations, support beyond ten innings, and redesigning the 50-game box-score limit. See [PLAN.md](PLAN.md) for detail.

- [Flat export data dictionary](DATA_DICTIONARY.md)
- [Parsing and calculation methodology](METHODOLOGY.md)
- [Recommendations for ThemeBoy/SportsPress](SPORTSPRESS_RECOMMENDATIONS.md)

## License and credit

Created by [jsrichter](https://github.com/jsrichter). The project is intended to be released under the MIT License. A license file has not yet been added, so the repository should not be treated as formally MIT-licensed until one exists.
