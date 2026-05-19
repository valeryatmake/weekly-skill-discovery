# Weekly Skill Discovery

A Claude skill that scans Jira, Confluence, Slack, Gmail, and Google Calendar every Monday for repeatable work patterns, proposes 3–5 skill candidates, notifies you in Slack with a link back to the session, and builds any skill you approve.

## What's in this repo

- **[SKILL.md](SKILL.md)** — the skill itself. Drop it into your Claude skills folder. The `CONFIG` block at the top fills itself in on the first run.
- **[index.html](index.html)** — landing page describing the skill (deployed via Vercel).
- **[make-design.css](make-design.css)** — Make 2026 design-system tokens used by the landing page.

## Install

1. Copy the `weekly-skill-discovery/` folder (containing `SKILL.md`) into your Claude skills directory.
2. Make sure these MCPs are connected in Claude's settings: **Atlassian** (Jira + Confluence), **Slack**, **Google Calendar**, **Gmail**.
3. In Claude's Cowork tab, start a new task, run `/schedule`, point it at your copy of `SKILL.md`, and set it to run every Monday at 9 AM. Enable **Keep awake**.
4. Hit **Run now** once. Claude detects the skill isn't configured for you yet and asks four quick questions — name, timezone, skills-folder path, Slack channel. After that, it runs automatically every week.

## How it works

| Phase | What happens |
|---|---|
| **00 — Identity check** | First run only. Claude asks for your name, timezone, skills path, and Slack channel, then writes them into `SKILL.md`. |
| **01 — Discovery** | Scans the last 7 days across all five sources in parallel, deduplicates against skills you already have, and ranks patterns by estimated weekly time cost. |
| **02 — Notify** | Posts a Slack summary with a link back to the session; full candidate details land in the Claude conversation. |
| **03 — Build** | Reply `build #N` and Claude writes the `SKILL.md`, schedules it, and confirms — typically under a minute. |

See the [landing page](https://weekly-skill-discovery.vercel.app) for the full walkthrough.

## License

Share freely. Built with Claude.
