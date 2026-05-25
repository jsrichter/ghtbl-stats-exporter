# GHTBL Stats Exporter

A browser-based tool for turning a WordPress/SportsPress XML export into baseball box scores, cleaned flat stat files, and exploratory stat summaries.

## What It Does

- Uploads a WordPress XML export locally in the browser.
- Parses SportsPress teams, players, and game events.
- Generates game box scores with batting and pitching sections.
- Exports box scores to `.xlsx`.
- Exports a cleaned flat stats file with game, season, date, team, player, and stat columns.
- Provides filtered dashboards and stat leaderboards for teams and players.

## Data Privacy

This is a static client-side app. Uploaded XML files are processed in your browser and are not sent to a server by this project.

The current `index.html` was checked for common sensitive values such as API keys, tokens, passwords, private local paths, and localhost URLs. No sensitive code or credentials were found.

The page does load public CDN assets:

- Tailwind CSS
- Tabler Icons
- Google Fonts
- SheetJS/XLSX

## Input Requirements

Upload a single WordPress XML export containing SportsPress data:

- `sp_event`
- `sp_player`
- `sp_team`

The tool expects SportsPress metadata such as:

- `sp_results`
- `sp_players`
- `sp_status`
- `sp_metrics`

## Hosting On GitHub Pages

This project can be hosted directly with GitHub Pages because it is a static HTML app.

1. Make sure the main file is named `index.html`.
2. Push the repository to GitHub.
3. Open the repository settings.
4. Go to **Pages**.
5. Set source to **Deploy from a branch**.
6. Select the `main` branch and `/root` folder.
7. Save.

The site will be available at:

```text
https://<username>.github.io/<repo-name>/
```

## Local Use

Open `index.html` directly in a browser, then upload your XML export.

No build step is required.

## Notes

- Regular season vs playoff filtering is inferred from game season, league, and title text.
- Flat stats generation happens only during export to avoid lag.
- Leaderboards use minimum qualification thresholds where applicable.
