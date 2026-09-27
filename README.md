# GM TS Priority Queue

A static dashboard for tracking weekly troubleshoot (TS) compliance across all GMs — eligibility buckets, PNL/RTO movement, roster changes, and the Zero GMV Monday task.

This is a plain static site: `index.html` + JSON files in `data/`. No build step, no framework, no server code. It fetches its data from `data/*.json` on load.

## How data gets refreshed

Right now, data refresh still happens through Claude (Metabase + Google Sheets access aren't independently available to this repo yet). The workflow is:

1. Claude pulls fresh data from Metabase + the roster Google Sheet.
2. Claude regenerates the three files in `data/`.
3. Claude commits and pushes those files to this repo.
4. Vercel auto-deploys on every push to `main` — the live site updates within ~30 seconds.

**This still requires the Claude session that runs the refresh to be active** (same constraint as before, just with a nicer frontend and working downloads now). To make refreshes fully independent of any laptop/session being open, the backend would need to run as real server code (e.g. a Vercel Cron Job or GitHub Action) with its own direct API credentials:
- A Metabase **Personal API Key** (Admin → Settings → API Keys)
- Either public read access to the roster Google Sheet, or a Google service account key

Once those exist, the refresh step can move to a scheduled serverless function instead of depending on Claude being asked to run it.

## Local development

No build step needed — just serve the folder:

```bash
python3 -m http.server 8080
```

Then open `http://localhost:8080`.

## Deploying

This repo is meant to be connected to Vercel via its GitHub integration — every push to `main` triggers a new deployment automatically. No `vercel.json` or build configuration is needed for a plain static site.

## Data files

- `data/gm_sellers.json` — one entry per GM (keyed by a slug), each holding that GM's sellers with their eligibility, bucket, PNL/spend/RTO, TS status, and movement history.
- `data/weekly_compliance.json` — one entry per ISO week (`"2026-W39"` etc.) with eligible/done/manual/auto counts per GM, plus Zero GMV Monday-task compliance for that week.
- `data/roster_changes.json` — one entry per day the roster was diffed, recording which sellers were added/removed since the previous day's snapshot.
