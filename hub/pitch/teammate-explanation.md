# Hey team — here's what I built and why it helps you

I put together a coordination hub for our project. It's a live dashboard that shows where every team stands at a glance — what's done, what's in progress, what's blocked. Bill can see it too, which means our work is visible without anyone having to chase us for updates.

Here's the short version of how it works:

**You keep a single small file in your repo** — a `status.json` that you update whenever something changes. Push it, and your card on the hub updates automatically. Takes two minutes. No login, no new account, no learning a new tool.

**The hub is public and live at:**
https://glenn-lewis-hub.github.io/webd3100-team-hub/

To get your team card showing on the hub, just send me the raw URL to your `status.json` on GitHub and I'll plug it in. Your card appears within minutes.

---

## What the hub does for you

- Your progress is always visible without you having to repeat yourself in meetings
- Blockers surface at the top of every weekly meeting — low pressure, just a conversation starter
- Meeting notes and decisions get logged so nothing gets lost between sessions
- A progress feed lets you post quick updates any time with a timestamp — useful when you finish something at 11pm and want the team to know
- A team board for quick messages so you're not relying on everyone being on the same Discord

## What we agreed on as a team

- **Weekly minimum** — update your status at least once a week, more whenever you have something to share
- **Blockers** — if you're stuck, flag it. It goes to the top of the meeting agenda. No pressure, just so we can help
- **Done means done** — when you mark a component complete, it means: working code, documented, accessibility checked with Edward, and ready for the next team to pick up at Week 9
- **Disagreements** — if two teams genuinely can't agree on a direction, both sides post a short video and the class votes. Last resort, not the default
- **Edward** — check in with him early at each milestone, not just at the end

## What I need from you

1. Create a `status.json` in your repo using the template below
2. Send me the raw GitHub URL to that file
3. Update it whenever something changes

That's it. The rest takes care of itself.

---

## status.json template

```json
{
  "team": "Team N — Your Team Name",
  "owner": "Your Name",
  "week": 1,
  "status": "in_progress",
  "done": [],
  "in_progress": ["What you're working on right now"],
  "next": ["What's coming up"],
  "blockers": [],
  "notes": "",
  "updated": "2026-10-01"
}
```

**Status values:** `not_started` · `in_progress` · `review` · `blocked` · `complete`

---

Questions? Message me on Teams or bring it up at the next meeting. Happy to walk anyone through it.
