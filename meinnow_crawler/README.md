# mein-now course tracker

Tracks your brand's course listings on [mein-now.de](https://mein-now.de/weiterbildungssuche/)
by pulling structured data from the mein-NOW / KURSNET search backend (the same
API the public search page uses — no account or scraping of rendered HTML needed).

## What it captures

Per run (daily), for the providers and keywords you configure:

- **Inventory** — every course listed under your provider name(s): title, locations,
  start dates, cost band, course format (Vollzeit/Teilzeit, Combined Learning, …),
  number of sessions, and the full description text (your keyword source).
- **Ranking / visibility** — for each tracked search keyword, the position your
  courses occupy in the result list. Lower rank number = more visible.
- **Changes** — a daily diff: which listings are new, which dropped off.

### What it does NOT capture

Clicks, impressions, and engagement are **not available to any crawler** — they
live in the platform's private analytics. To track clicks, use your mein-NOW
provider dashboard or Google Analytics / Search Console on your own pages.

## Setup

1. Edit [`config.json`](config.json):
   - `providers`: your brand name(s) exactly as shown on mein-now.de (matching is
     case-insensitive substring — `"GFN"` matches `"GFN GmbH"`).
   - `ranking_keywords`: the search terms prospective students would type.
2. Run locally: `pip install -r requirements.txt && python crawl.py`
3. Outputs land in `data/`.

## Hosting (free, GitHub Actions)

The workflow at `.github/workflows/crawl.yml` runs the crawler daily and commits
the new snapshots back to the repo — $0/month, full history in git.

- Push this repo to GitHub.
- Settings → Actions → General → Workflow permissions → enable **Read and write**.
- The job runs daily at 05:15 UTC; trigger it manually any time from the **Actions**
  tab → *mein-now course crawl* → *Run workflow*.

## Output files

| File | Contents |
|------|----------|
| `data/snapshots/inventory_YYYY-MM-DD.csv` | One row per course — daily snapshot |
| `data/snapshots/inventory_YYYY-MM-DD.jsonl` | Same, full detail per line |
| `data/rankings.csv` | Appended every run: `date, keyword, provider, course_id, title, rank, total_results` — your visibility over time |
| `data/changes/changes_YYYY-MM-DD.md` | Human-readable diff vs the previous snapshot |

Open `rankings.csv` in Excel/Sheets and pivot by keyword to chart rank trends.

## API notes

- Backend: `https://rest.mein-now.de/now-prod/suche/pc/v1/bildungsangebot`
- Auth header: `X-API-Key: infosysbub-nowsuche` (a public client id read from the
  site config — not a secret). If crawls start returning 404, the host or client id
  has changed; re-read it from `https://mein-now.de/weiterbildungssuche/assets/config.json`
  and update `config.json`.
