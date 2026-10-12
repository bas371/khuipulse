# Khui Pulse (self-hosted)

A static dashboard. `index.html` loads `data.json` when it opens, so refreshing
the dashboard means replacing `data.json`. Nothing else changes.

## Files
- `index.html`: the dashboard (same logic as the Claude artifact, data loaded from `data.json`)
- `data.json`: all numbers. Top-level keys: daily, camp, aset, ads, gg, meta, biz, markets, targets, pin, ref, pdaily, chan
- `scripts/refresh.py`: rebuilds `data.json`. Keys it doesn't touch are kept as they are
- `.github/workflows/refresh.yml`: runs refresh.py on a schedule and commits the result

## Setup

### 1. GitHub
1. Create a private repo at https://github.com/new (e.g. `khui-pulse`). Leave "Add a README" unticked if you upload this one.
2. On the empty repo page click "uploading an existing file".
3. Drag in the CONTENTS of this folder (index.html, data.json, README.md, scripts/, .github/), not the outer folder.
   - `.github` is hidden. Mac: Cmd+Shift+. in Finder. Windows: File Explorer > View > Show > Hidden items.
   - Fallback: Add file > Create new file, name it `.github/workflows/refresh.yml`, paste the contents, commit.
4. Commit changes. The repo root must show index.html, data.json, scripts/ and .github/.

### 2. Cloudflare Pages (free hosting)
1. https://dash.cloudflare.com/ > Workers & Pages > Create > Pages > Connect to Git.
2. Pick the repo. Build command: empty. Output directory: `/`. Save and Deploy.
3. You get a `*.pages.dev` link. Every commit to the repo redeploys it.

### 3. Make it private
1. Pages project > Settings > General > Access policy > Enable.
2. https://one.dash.cloudflare.com/ > Access: add an Allow policy for your team's emails.
Without this the page is public.

### 4. Data source (Google Sheet)
1. Put each table in its own tab. Looker Studio > Sheets exports can feed it.
2. Sheets > File > Share > Publish to web > pick the tab > CSV > copy the link.
3. GitHub repo > Settings > Secrets and variables > Actions > New repository secret:
   name `DAILY_CSV_URL`, value = the link.

### 5. Wire up the refresh
1. Edit `scripts/refresh.py`: uncomment the `fetch_daily()` lines and match columns to your sheet.
   Repeat the pattern for camp / aset / ads / biz.
2. Repo > Actions > enable workflows > `refresh-khui-pulse` > Run workflow to test.
3. On success it commits a new data.json and Cloudflare redeploys in about a minute.
   Schedule: 08:00 and 14:00 Bangkok. Change the `cron` line in refresh.yml (it is UTC).

### 6. Check
Open the pages.dev link and compare against the original artifact.
"Could not load data.json" means data.json is not in the repo root.

## Notes
- This version only works on an outside host. A published Claude artifact cannot fetch a separate file.
- The refresh depends on whatever feeds the Sheet.
- Free-tier limits change, so check Cloudflare and GitHub before relying on them.
