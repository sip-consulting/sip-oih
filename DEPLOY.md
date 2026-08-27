# Deploying SIP-OIH with automatic updates (free, no server to maintain)

This gets you a permanent link — something like
`https://sipconsulting.github.io/sip-oih/` — that always shows the latest
opportunities, refreshed automatically every 10 hours, with no VPS, no
FastAPI hosting, and no monthly bill.

**How it works:** GitHub Actions (free for public repos, and free for
private repos up to a generous monthly minute allowance) runs the sync
script on a timer, commits the refreshed `data/opportunities.json` back to
the repo, and GitHub Pages serves the dashboard, which fetches that JSON
file on load. Opening the link always shows whatever the last scheduled run
found.

## What's in this package

```
index.html                       ← the dashboard (same file as sip_oih.html)
data/opportunities.json          ← current data snapshot (overwritten by every sync)
scripts/sync_and_export.py       ← fetches BrightSpyre + ReliefWeb, scores, writes the JSON
.github/workflows/sync.yml       ← the schedule: runs every 10 hours
```

## Step-by-step setup (about 10 minutes)

### 1. Create the GitHub repository
- Go to github.com, click **New repository**.
- Name it something like `sip-oih`. Public is simplest (free, unlimited
  Actions minutes); private also works with GitHub's free minute allowance.
- Don't initialize with a README — you're uploading these files instead.

### 2. Upload these files
Easiest way without using the command line: on the new repo's page, click
**"uploading an existing file"**, drag in this whole folder's contents
(keeping the folder structure — `index.html` at the root, `data/`,
`scripts/`, and `.github/workflows/` as subfolders), and commit.

If you're comfortable with git:
```bash
cd sip-oih-deploy
git init
git add .
git commit -m "Initial SIP-OIH deployment"
git branch -M main
git remote add origin https://github.com/YOUR-USERNAME/sip-oih.git
git push -u origin main
```

### 3. Enable GitHub Pages
- In the repo, go to **Settings → Pages**.
- Under "Build and deployment," set **Source** to "Deploy from a branch."
- Branch: `main`, folder: `/ (root)`. Save.
- GitHub gives you the live URL here — usually
  `https://YOUR-USERNAME.github.io/sip-oih/` — within a minute or two.

That URL is what you open going forward. Bookmark it, share it with the
team — that's the "just open the link" experience you asked for.

### 4. Let the sync workflow run
- It's already scheduled for every 10 hours (`.github/workflows/sync.yml`).
- To see it work immediately rather than waiting: go to the repo's
  **Actions** tab → **Sync opportunities** → **Run workflow** (the
  `workflow_dispatch` trigger already included lets you do this manually).
- It fetches from ReliefWeb's API and BrightSpyre, scores everything, and
  commits the updated `data/opportunities.json`. GitHub Pages picks up the
  change automatically on the next page load — no redeploy step needed.

### 5. Confirm it's live
Open your Pages URL. If the sync has run at least once, the dashboard will
load `data/opportunities.json` instead of the embedded snapshot — you can
tell because the "Last sync" label switches to "Live — updated by
scheduled sync." If it still says "Embedded snapshot," the workflow hasn't
completed a run yet — check the Actions tab for errors.

## Adding more sources later

Add a new fetch function to `scripts/sync_and_export.py` (following the
same pattern as `fetch_reliefweb()` / `fetch_brightspyre()`), call it in
`main()`, and commit — the next scheduled run will include it. This is the
same code structure as the standalone backend package built earlier
(`sip_mis_backend.zip`); this deployment script is a trimmed,
GitHub-Actions-friendly version of the same connectors.

## Why not just host the FastAPI backend?

That's a legitimate alternative — Render, Railway, or a small VPS running
the FastAPI service from `sip_mis_backend.zip` would work too, and gives
you a real database instead of a JSON file, which matters once the team
needs shared pipeline state (shortlisting, stage changes) rather than
per-browser localStorage. This GitHub Pages approach is the faster, free
starting point; moving to the full backend is Phase 3 in the roadmap
(`SIP-OIH-README.md`) and doesn't require rebuilding the frontend — just
pointing the same `fetch('./data/opportunities.json')` call at a real API
URL instead.

## Limitations worth knowing

- **GitHub Actions cron isn't second-precise.** "Every 10 hours" can drift
  by a few minutes depending on GitHub's scheduler load — fine for this use
  case, not fine if you needed exact timing.
- **Pipeline stage changes and partner entries still live in each
  visitor's own browser** (localStorage), not centrally. If two people open
  the link, they don't see each other's shortlisting. Shared state requires
  the real backend + database (Phase 3).
- **Scraping reliability**: BrightSpyre's HTML structure can change without
  notice; if a sync run fails, check the Actions tab logs — the dashboard
  will simply keep showing the last successful snapshot rather than break.
