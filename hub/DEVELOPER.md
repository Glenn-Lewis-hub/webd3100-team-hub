# NextStep Team Hub — Developer Documentation

Complete guide to understanding, recreating, and extending the hub in any developer's environment.

---

## What this is

A static team coordination dashboard deployed to GitHub Pages. No backend, no database, no build step. All data is driven by JSON files — some local to this repo, some fetched at runtime from each team's own GitHub repo. The hub reads, renders, and auto-refreshes. Teams update their JSON files via git push; the hub reflects changes within ~5 minutes.

---

## Prerequisites

| Tool | Version | Purpose |
|------|---------|---------|
| Git | Any recent | Cloning, committing, pushing |
| GitHub account | — | Repo hosting, Pages, Discussions |
| GitHub CLI (`gh`) | 2.x+ | Repo creation, Pages config, API calls |
| A browser | Modern | Testing (hub uses fetch + ES2020) |
| A local HTTP server | Any | Local testing (file:// blocks fetch()) |

**No Node, no npm, no build tools required.**

---

## Repository structure

```
/
├── .github/
│   └── workflows/
│       └── deploy-hub.yml      ← GitHub Actions: deploys /hub to Pages on push to main
├── hub/
│   ├── index.html              ← Entire dashboard (markup + CSS + JS, self-contained)
│   ├── pitch/                  ← Pitch documents (teammate, teacher, class)
│   └── data/
│       ├── teams.json          ← Team config: IDs, names, colours, status_url per team
│       ├── roadmap.json        ← Project phases with milestones and status
│       └── updates.json        ← Weekly update log: video links, summaries, decisions
├── team1-shell/
│   ├── status.json             ← Team 1 status (or in their own repo if separate)
│   └── feed.json               ← Team 1 progress feed entries
├── team2-profile/
│   ├── status.json
│   └── feed.json
├── team3-matching/
│   ├── status.json
│   └── feed.json
├── team4-nextstep/
│   ├── status.json
│   └── feed.json
├── team5-lookforward/          ← Glenn's component
│   ├── status.json
│   └── feed.json
├── shared/
│   ├── data/                   ← Instructor-provided data files
│   └── design/                 ← Design tokens, style guide
├── CODEOWNERS                  ← Folder ownership rules
├── PLAN.md                     ← Team workflow plan
└── README.md                   ← Project overview
```

---

## Standing up from scratch

### 1. Clone or fork

```bash
git clone https://github.com/Glenn-Lewis-hub/webd3100-team-hub.git
cd webd3100-team-hub
```

Or create fresh:

```bash
mkdir my-team-hub && cd my-team-hub
git init
git config user.email "your-noreply@users.noreply.github.com"
git config user.name "your-github-username"
```

### 2. Create the GitHub repo

```bash
gh repo create your-username/my-team-hub --public \
  --description "Team coordination hub"
git remote add origin https://github.com/your-username/my-team-hub.git
git branch -M main
```

### 3. Enable GitHub Pages (Actions source)

```bash
gh api repos/your-username/my-team-hub/pages \
  --method POST -f build_type=workflow
```

### 4. Enable GitHub Discussions

```bash
gh api repos/your-username/my-team-hub \
  -X PATCH -f has_discussions=true
```

### 5. Install the giscus GitHub App

Go to **https://github.com/apps/giscus** in a browser.
Click Install → select your repo → confirm.

This is the only step that cannot be done from the CLI.

### 6. Get your repo's node ID and Discussion category ID

```bash
gh api graphql -f query='
{
  repository(owner: "your-username", name: "my-team-hub") {
    id
    discussionCategories(first: 10) {
      nodes { id name }
    }
  }
}'
```

Note the repo `id` (starts with `R_`) and the `id` of the category you want (e.g. General, starts with `DIC_`).

### 7. Update hub/index.html with your repo IDs

Find these constants near the top of the `<script>` section:

```js
const GISCUS_REPO     = 'your-username/my-team-hub';
const GISCUS_REPO_ID  = 'R_YOUR_REPO_ID';
const GISCUS_CATEGORY = 'General';
const GISCUS_CAT_ID   = 'DIC_YOUR_CATEGORY_ID';
```

Replace with your values.

### 8. Update hub/data/teams.json

Add each team's `status_url` as they share their raw GitHub links:

```json
{
  "teams": [
    {
      "id": "team1-shell",
      "name": "App Shell & Navigation",
      "color": "#3b82f6",
      "status_url": "https://raw.githubusercontent.com/teammate/their-repo/main/status.json"
    }
  ]
}
```

Leave `status_url` as `""` for teams not yet connected — they'll show a "Waiting for link" card.

### 9. Push and deploy

```bash
git add .
git commit -m "Initial hub setup"
git push -u origin main
```

GitHub Actions runs automatically. Hub is live at:
`https://your-username.github.io/my-team-hub/`

---

## Local development

The hub uses `fetch()` which is blocked on `file://` URLs by browser security. You need a local HTTP server.

**Python (simplest):**
```bash
python -m http.server 8090 --directory hub
# Open http://localhost:8090
```

**VS Code Live Server:**
Right-click `hub/index.html` → Open with Live Server.

**Node (if installed):**
```bash
npx serve hub -l 8090
```

When running locally, fetches to `raw.githubusercontent.com` still go to GitHub — your local changes to status.json won't be reflected until pushed and cached (up to 5 min).

---

## Data file schemas

### hub/data/teams.json

```json
{
  "teams": [
    {
      "id": "team1-shell",
      "name": "Team 1 — App Shell & Navigation",
      "color": "#3b82f6",
      "status_url": "https://raw.githubusercontent.com/owner/repo/main/status.json"
    }
  ]
}
```

`color` accepts any valid CSS colour. `status_url` is the raw GitHub URL to the team's `status.json`. The feed URL is automatically derived by replacing `status.json` with `feed.json` in the same path.

### teamN/status.json

```json
{
  "team": "Team 1 — App Shell & Navigation",
  "owner": "Name",
  "week": 1,
  "status": "in_progress",
  "done": ["Item 1", "Item 2"],
  "in_progress": ["Current work"],
  "next": ["Upcoming item"],
  "blockers": [],
  "notes": "Optional freetext note",
  "updated": "2026-10-01"
}
```

**Status values:** `not_started` · `in_progress` · `review` · `blocked` · `complete`

**Required fields** (hub validates these — malformed files get an error card):
`team` (string), `status` (string), `updated` (string)

**Stale threshold:** if `updated` is more than 7 days ago, the card shows an amber border and "Stale" badge.

### teamN/feed.json

```json
{
  "entries": [
    {
      "date": "2026-10-01T15:42:00Z",
      "author": "Name",
      "team": "team1-shell",
      "team_name": "App Shell & Navigation",
      "message": "What was completed or started."
    }
  ]
}
```

Add new entries to the array, newest last. The hub sorts by `date` descending automatically.

### hub/data/roadmap.json

```json
{
  "phases": [
    {
      "id": "phase1",
      "name": "Foundation",
      "weeks": "1–3",
      "description": "One-sentence description.",
      "status": "in_progress",
      "milestones": ["Milestone 1", "Milestone 2"]
    }
  ]
}
```

### hub/data/updates.json

```json
{
  "updates": [
    {
      "week": 1,
      "date": "2026-10-01",
      "title": "Week 1 — Kickoff",
      "video_url": "https://youtube.com/watch?v=...",
      "summary": "One-paragraph summary.",
      "decisions": ["Decision 1", "Decision 2"]
    }
  ]
}
```

`video_url` can be empty string — hub shows "Video not yet posted" placeholder.

---

## CODEOWNERS

```
# Syntax: /path/ @github-username
/hub/                   @your-username
/shared/                @your-username
/team1-shell/           @teammate1-github
/team2-profile/         @teammate2-github
```

Requires branch protection on `main` with "Require review from code owners" enabled. Without branch protection, CODEOWNERS is documentation only — it doesn't enforce anything.

**Enable branch protection:**
```bash
gh api repos/your-username/my-team-hub/branches/main/protection \
  --method PUT \
  --input - <<'EOF'
{
  "required_status_checks": null,
  "enforce_admins": false,
  "required_pull_request_reviews": {
    "required_approving_review_count": 1,
    "require_code_owner_reviews": true
  },
  "restrictions": null
}
EOF
```

---

## GitHub Actions workflow

```yaml
# .github/workflows/deploy-hub.yml
name: Deploy Hub to GitHub Pages

on:
  push:
    branches: [main]

permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: pages
  cancel-in-progress: true

jobs:
  deploy:
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/configure-pages@v4
      - uses: actions/upload-pages-artifact@v3
        with:
          path: ./hub           # Only the /hub folder is deployed
      - uses: actions/deploy-pages@v4
        id: deployment
```

Only the `/hub` folder is served — team status folders, PLAN.md, etc. are not public-facing.

---

## Hub JS architecture (hub/index.html)

All logic lives in a single `<script>` block at the bottom of `index.html`. Key constants and functions:

| Symbol | Type | Purpose |
|--------|------|---------|
| `GISCUS_*` | constants | Repo and category IDs for the discussion board |
| `STALE_DAYS` | constant | Days before a card is marked stale (default 7) |
| `REFRESH_MS` | constant | Auto-refresh interval in ms (default 300000 = 5 min) |
| `TEAM_COLORS` | object | Hex colour per team ID — fallback if teams.json lacks colour |
| `fetchJSON(url)` | function | Fetch + JSON parse with cache-bust query param |
| `validateStatus(data)` | function | Returns false if required fields missing |
| `isStale(dateStr)` | function | Returns true if date is older than STALE_DAYS |
| `relTime(dateStr)` | function | "2h ago", "3d ago", etc. |
| `esc(s)` | function | XSS-safe escaping via textContent round-trip |
| `deriveFeedUrl(url)` | function | Replaces status.json with feed.json in URL |
| `loadAll()` | async function | Master loader — teams, feed, board tabs, roadmap, updates |
| `loadBoard(teamId)` | function | Injects giscus script for a specific team |

`loadAll()` runs on boot and every `REFRESH_MS` milliseconds via `setInterval`.

All team status fetches use `Promise.allSettled()` — failures are isolated per card, the rest of the hub renders normally.

---

## Extending the hub

### Add a new team

1. Add an entry to `hub/data/teams.json`
2. Create `teamN-name/status.json` and `teamN-name/feed.json`
3. Add the folder to `CODEOWNERS`
4. Push — hub picks it up automatically

### Add a new roadmap phase

Edit `hub/data/roadmap.json` — add an object to the `phases` array.

### Post a weekly update

Edit `hub/data/updates.json` — prepend a new object to the `updates` array. Set `video_url` once the video is posted; leave as `""` until then.

### Change the stale threshold

Edit the `STALE_DAYS` constant in `hub/index.html`.

### Change the refresh interval

Edit the `REFRESH_MS` constant in `hub/index.html`.

### Add a new giscus board category

By default all team boards use the `General` Discussion category. To give each team its own category:
1. Create categories in GitHub Discussions (Repo → Discussions → Manage categories)
2. Get each category's ID via the GraphQL query in step 6 above
3. Update `loadBoard()` to use a per-team category ID instead of `GISCUS_CAT_ID`

---

## Known limitations

| Limitation | Detail |
|------------|--------|
| 5-min cache lag | raw.githubusercontent.com CDN caches files for ~5 minutes. Status updates won't appear instantly. |
| No write capability | Hub is read-only. Teams update their JSON via git — no in-hub editing. |
| Giscus requires app install | One-time browser action per repo. Cannot be scripted. |
| file:// blocked | Must use a local HTTP server for development. |
| No auth | The hub is public. Anyone with the URL can see all team status. Appropriate for a class project; not for sensitive work. |
