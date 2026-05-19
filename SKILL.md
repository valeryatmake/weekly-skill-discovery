---
name: weekly-skill-discovery
description: Scans Jira, Confluence, Slack, Gmail, and Google Calendar for repeatable work patterns, proposes 3–5 skill candidates, notifies via Slack with a link back to this session, then builds any skill you approve.
---

<!--
  CONFIG — populated automatically on first run. Do not edit manually.

  USER_NAME:      YOUR_NAME
  USER_TIMEZONE:  YOUR_TIMEZONE
  SCHEDULED_PATH: YOUR_SCHEDULED_PATH
  SLACK_CHANNEL:  YOUR_SLACK_CHANNEL

  Prerequisites (MCP connections required):
  - Atlassian MCP (Jira + Confluence)
  - Slack MCP
  - Google Calendar MCP
  - Gmail MCP
-->

**Before running any phase:** read the CONFIG block in the HTML comment at the top of this file and resolve the four variables — `USER_NAME`, `USER_TIMEZONE`, `SCHEDULED_PATH`, `SLACK_CHANNEL` — to their actual values. Use those values everywhere these placeholders appear below.

You are running the weekly skill discovery for {USER_NAME}. Timezone: {USER_TIMEZONE}. Today's date is available from context.

Your job: find repeatable manual work that should be automated as a Claude skill. Run Phase 1 automatically on schedule. Stop after Phase 2 and wait for {USER_NAME} to choose a candidate to build.

---

## Phase 0 — Identity check (runs every time)

Before doing anything else:

1. Read this SKILL.md file from disk and extract the CONFIG values.
2. Use the Atlassian MCP tool (tool name contains `atlassianUserInfo`) to get the current user's display name and account ID. Store both — the display name is used for the identity check, the account ID is used in Phase 1 Jira and Confluence queries.
3. Compare the current user's display name to `{USER_NAME}` from CONFIG.

**If they match** → skip the rest of Phase 0 and proceed directly to Phase 1.

**If they don't match** (this skill was copied from someone else, or hasn't been configured yet), enter setup mode:

1. Greet the user:
   > "Welcome! This skill hasn't been set up for you yet. I'll ask a few quick questions — this only happens once."

2. Ask the following questions one at a time, waiting for each answer:
   - "What's your full name?"
   - "What's your timezone? (e.g. Europe/Prague, America/New_York, Asia/Singapore)"
   - "What's the full path to the folder where your Claude skills live? (e.g. /Users/yourname/Documents/Claude/Scheduled)"
   - "Which Slack channel should I notify when I find new skill candidates? (e.g. #my-channel)"

3. Use the Edit tool to update this SKILL.md file — replace the four CONFIG values with the user's answers:
   - `USER_NAME` → their full name
   - `USER_TIMEZONE` → their timezone
   - `SCHEDULED_PATH` → their skills folder path
   - `SLACK_CHANNEL` → their Slack channel

4. Re-read this SKILL.md file from disk and re-resolve all four CONFIG variables to their newly written values. Use these updated values for all remaining phases.

5. Confirm:
   > "✅ All set! Running your first discovery scan now..."

6. Continue directly to Phase 1.

---

## Phase 1 — Discovery

### Step 1 — Build exclusion list

Read all files matching `{SCHEDULED_PATH}/*/SKILL.md`. Extract the `name` and `description` from each frontmatter block. These skills already exist — do not propose them again.

### Step 2 — Scan all five sources (run in parallel)

Use the Atlassian account ID retrieved in Phase 0 for all Jira and Confluence queries below.

**Slack** — use the Slack MCP tool (tool name contains `slack_search_public_and_private`)
- Query 1: `from:me after:YYYYMMDD` (7 days ago, ISO date) — your own sent messages
- Query 2: `is:saved` — your saved/bookmarked messages
- Look for: repeated phrases or questions you keep asking, messages that required manual research or synthesis, recurring topics you flag for later

**Google Calendar** — use the Google Calendar MCP tool (tool name contains `list_events`) on the `primary` calendar
- Fetch events from 7 days ago through 7 days ahead
- Look for: recurring meetings that have no existing prep or debrief skill, meetings that always result in similar manual follow-up actions

**Gmail** — use the Gmail MCP tool (tool name contains `search_threads`)
- Query: `in:sent after:YYYYMMDD` (7 days ago)
- Look for: emails where you typed similar content multiple times, recurring threads that required the same type of research or response structure

**Jira** — use the Atlassian MCP tool (tool name contains `searchJiraIssuesUsingJql`)
- JQL: `assignee = "[your account ID]" AND updated >= -7d ORDER BY updated DESC`
- Also try: `reporter = "[your account ID]" AND created >= -7d`
- Look for: issue types you create repeatedly, recurring investigation or documentation tickets, any manual status-update patterns

**Confluence** — use the Atlassian MCP tool (tool name contains `searchConfluenceUsingCql`)
- CQL: `contributor = "[your account ID]" AND lastModified >= now("-7d")`
- Look for: pages you update on a regular cadence, recurring documentation formats, pages that always require the same type of content refresh

### Step 3 — Synthesize and rank

From all sources, identify patterns that:
- Appear more than once OR across multiple tools
- Take meaningful time to do manually (>5 min each occurrence)
- Are not already covered by existing skills (from Step 1)
- Would be feasible to automate as a Claude skill

Rank by estimated weekly time cost. Select the top 3–5 candidates.

For each candidate, prepare:
- A short name
- The concrete pattern spotted (with a specific example from the data)
- What the skill would do
- Estimated time saved per week
- Which sources surfaced it

---

## Phase 2 — Notify and present

### Step 4 — Get session URL (best effort)

Try to get the URL of the current Claude session using the browser MCP tool (tool name contains `tabs_context_mcp`). If successful, use that URL in the Slack message. If unavailable or it fails, use the fallback text below.

### Step 5 — Send Slack notification

Use the Slack MCP tool (tool name contains `slack_send_message`) to post to `{SLACK_CHANNEL}`.

Slack message format (mrkdwn):

```
🔍 *Weekly Skill Discovery — [Weekday, Date]*
――――――――――――――――――――――
Found *[N]* repeatable patterns that could become skills:

*1. [Skill Name]* — [one-sentence description]
*2. [Skill Name]* — [one-sentence description]
[repeat]

――――――――――――――――――――――
👉 <[session URL]|Continue in Claude> to review details and build one.
```

If session URL was unavailable, replace the last line with:
`👉 Open Claude and continue the *Weekly Skill Discovery — [Date]* session to review and build.`

**Formatting rules:**
- Use `*bold*` and `_italic_` only
- Use `――――――――――――――――――――――` as dividers
- Use `<URL|link text>` for all hyperlinks

### Step 6 — Present full candidates in the Claude conversation

Display the full candidate list in this format:

---

## 🔍 Weekly Skill Discovery — [Date]

Here are the skill candidates I found from the past 7 days:

---

**1. [Skill Name]**
📋 *Pattern:* [concrete example of the repeating action, e.g. "You wrote Jira status update comments in 4 separate tickets using the same structure"]
💡 *What it would do:* [clear description of the skill's job]
⏱️ *Est. time saved:* ~[X] min/week
📡 *Sources:* [Jira / Slack / Calendar / etc.]

**2. [Skill Name]**
[same structure]

[repeat for all candidates]

---

Reply **"build #[N]"** (or use the skill name) and I'll create it.
To skip: close this session or say "skip".

---

> STOP here. Wait for {USER_NAME}'s reply before proceeding.

---

## Phase 3 — Build (triggered by user reply)

When {USER_NAME} selects a candidate:

### Step 7 — Clarify if needed

Ask at most 2 clarifying questions, only if the answer isn't obvious from context:
- Schedule: when should this run? (if it's a recurring skill)
- Output: where should results go? (Slack, Gmail, saved to file, etc.)

If the answer is obvious, skip and proceed directly.

### Step 8 — Write the SKILL.md

Create the file at `{SCHEDULED_PATH}/[skill-name]/SKILL.md`.

Follow the same structure as existing skills:
- YAML frontmatter with `name` and `description`
- Address {USER_NAME} by name at the top
- Numbered steps with clear, executable instructions
- Use the same Slack formatting conventions if the skill sends output there (use `――――――――――――――――――――――` as dividers, `*bold*` and `_italic_` only)
- End with: use the Slack MCP tool (tool name contains `slack_send_message`) to post to `{SLACK_CHANNEL}` if the skill outputs to Slack

### Step 9 — Schedule if recurring

If the skill should run on a schedule, use `mcp__scheduled-tasks__create_scheduled_task` with the appropriate cron expression and the path to the new SKILL.md.

### Step 10 — Confirm

Reply with:
```
✅ Skill created: `{SCHEDULED_PATH}/[skill-name]/SKILL.md`
[If scheduled:] Scheduled for [cadence, e.g. "every Tuesday at 8:00 AM CET"].
[If not scheduled:] Run it manually anytime from Claude Code.
```
