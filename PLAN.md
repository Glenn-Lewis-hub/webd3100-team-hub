# NextStep — Team Coordination Plan

## What this is

A shared project repo with a live team dashboard hosted on GitHub Pages. Every team gets their own folder. The hub reads each team's `status.json` and renders a unified view of where the project stands. Weekly update videos and meeting decisions are logged here too.

The whole repo — its commit history, the hub's evolution, the status updates across weeks — becomes a portfolio artifact showing how Glenn ran team coordination on a five-team class project.

---

## Repo structure

```
/
├── hub/                   ← Team dashboard (Glenn owns — GitHub Pages entry point)
│   ├── index.html
│   └── data/
│       ├── updates.json   ← Weekly video log
│       └── roadmap.json   ← Project phases
├── team1-shell/           ← App Shell & Navigation
│   └── status.json
├── team2-profile/         ← Profile & Intake
│   └── status.json
├── team3-matching/        ← Career Matching Results
│   └── status.json
├── team4-nextstep/        ← Next Step Recommendation
│   └── status.json
├── team5-lookforward/     ← Lookforward & Outcome Projection (Glenn)
│   └── status.json
├── shared/
│   ├── data/              ← Instructor-provided data files go here
│   └── design/            ← Shared tokens, style guide, component decisions
├── CODEOWNERS             ← Folder ownership rules
└── PLAN.md                ← This document
```

---

## How teams update their status

1. Clone the repo once. Pull before every session (`git pull origin main`).
2. Create or switch to your personal branch: `git checkout -b team5-lookforward` (do this once).
3. Edit only your team's `status.json`. Do not touch other folders.
4. Commit and push to your branch: `git push origin team5-lookforward`.
5. Open a pull request from your branch to `main`.
6. The CODEOWNERS rule means Glenn reviews PRs that touch `/hub/` and `/shared/`, and each team reviews their own. A merge to main re-deploys the hub automatically.

**Rule:** your branch can be named anything, but your commits should only ever touch your own folder. If you need something from shared/, raise it in a meeting and Glenn will update it.

---

## Status JSON schema

Each team maintains one file: `teamN-name/status.json`.

```json
{
  "team": "Team 5 — Lookforward",
  "owner": "Glenn Lewis",
  "week": 1,
  "status": "not_started",
  "done": [],
  "in_progress": [],
  "next": [],
  "blockers": [],
  "notes": "",
  "updated": "2026-10-01"
}
```

**Status values:** `not_started` · `in_progress` · `review` · `blocked` · `complete`

Update `week` to the current course week and `updated` to today's date each time you push.

---

## Weekly update rhythm

Every week Glenn records a short video (~5 min) covering:
- Where each team ended the week
- What the priority is for the coming week
- Any cross-team dependencies or decisions needed

The video link goes into `hub/data/updates.json`. Teams watch it at the start of the week before touching any code.

---

## Meeting log

Key decisions from team meetings go into `hub/data/roadmap.json` under the relevant phase. Meeting notes that don't fit a phase can be added as a `meetings` array in `updates.json`. The rule is: if a decision was made that affects how someone builds their component, it gets written down here.

---

## GitHub Pages setup (Glenn to do once)

1. Push the repo to GitHub.
2. In repo Settings → Pages → Source: `main` branch, `/hub` folder.
3. GitHub will give a URL like `https://Glenn-Lewis-hub.github.io/career-quiz/`.
4. Share that URL with all teammates — that's their read-only view of the hub.
5. Add branch protection to `main`: require PR + at least one review, no direct pushes.

---

## Portfolio angle

The repo tells the whole story in git history:
- Week 1: skeleton scaffold and team contracts
- Weeks 2–8: weekly status commits, hub evolving, roadmap filling in
- Week 9: handoff state — component handed off cleanly enough for another team to pick up
- Week 9–end: second component work

Present it as: "I designed and ran the coordination system for a five-team class project — here's how the hub evolved, here's the weekly cadence I built, here's the handoff doc."

---

## Handoff standard (Week 9 requirement)

Before Week 9 switch, the outgoing team must:
- Set `status: "complete"` in their `status.json`
- Add a `handoff_notes` field listing: what's done, what's in progress, any gotchas, how to run the component locally
- Push a final PR to main

The incoming team's first PR should update `owner` to their name.
