---
name: redmine-operations
description: Run Redmine operations via MCP (issues, projects, time entries, batch log time). Use when the user mentions Redmine, tickets, issues, projects, operational logs, or logging time.
---

# Redmine operations

Use **both** MCP servers from `.cursor/mcp.json`:

- **`redmine`**: team Westeros SSE gateway for standard Redmine operations.
- **`redmine-agent`**: local `npm run mcp` server with `redmine_agent_log_time`, `redmine_agent_create_issue`, `redmine_agent_batch_create_issues`, `redmine_agent_get_all_users`, `redmine_agent_list_permissions`.

| Server | Tool | Use for |
|--------|------|---------|
| **`redmine`** | Team SSE tools | Issues, projects, reads, updates |
| **`redmine-agent`** | `redmine_agent_log_time` | Batch log time + Billable Hours |
| **`redmine-agent`** | `redmine_agent_get_config` | Live trackers, activities, priorities, statuses (refresh with `force: true`) |
| **`redmine-agent`** | `redmine_agent_list_permissions` | User/role permission checks |

Do **not** route full issue creation here; use `.cursor/skills/redmine-ticket-creation/SKILL.md` and `redmine_agent_create_issue`.

## Tool routing

| User intent | Tool |
|----|---|
| Log time / batch timelog / multiple issues or dates | **`redmine_agent_log_time`** |
| Create issue / new task / ticket with full intake | **`redmine_agent_create_issue`** (see ticket-creation skill) |
| Batch create from spreadsheet / Google Sheet | **`redmine_agent_batch_create_issues`** — **must** follow batch-ticket-creation skill (preview + user confirm; AskQuestion for wrong assignees) |
| List assignable users (your project memberships) for assignee matching | **`redmine_agent_get_all_users`** (optional `projectId`) |
| What permissions do I have / roles / forbidden ops | **`redmine_agent_list_permissions`** |
| Get my time entries / logs | `get_my_time_entries` (team `redmine`) |
| My issues | `get_my_issues` (team `redmine`) |
| Get/search/update issues, projects, wiki | team `redmine` tools |
| Single-issue log (no batch) | Prefer **`redmine_agent_log_time`**; team `log_time` only if user insists |
| Everything else | team `redmine` |

## Required input discipline

- Use AskQuestion when a tool response has `needsInput` plus selectable options.
- When `questions[]` exists and `data.batchMode` is `true`, ask **all** questions in **one** AskQuestion.
- When `data.step` is `activities`, options are in `questions[]` (from cache). Show AskQuestion, then retry with `activityBatchId` + `pairActivities`.
- Do not call `redmine_agent_get_config` during timelog unless activities look stale.
- Never assume issue ID, hours, dates, comment text, or activity unless the user selected them.
- **Batch create:** follow batch-ticket-creation skill — map_columns / fix_column_mapping → preview tree → AskQuestion **yes/no** → then `proceed`. Never auto-fix assignees or columns; never upload on first call. Resolve sheet **Type** column via `redmine_agent_get_config` / `data.trackerOptions`; Task and Bug rows get **Steps to Reproduce** (comments → description → subject → `"..."`). Bug rows fail without create-time Steps to Reproduce; failed rows are not written back to the sheet.

## Batch log time — one screen per step

When `data.batchMode` is true, show **all** `questions[]` in **one** AskQuestion.

**Activities (Step 4):** Options come from the **server config cache** — loaded when the MCP server connects (`loadRedmineConfig`, 5‑minute TTL). Timelog does **not** call `redmine_agent_get_config` or Redmine again for the list; `questions[].options` is built from that cache. Use `redmine_agent_get_config` only if you need a manual refresh (`force: true`).

Before save, present a concise preview schema to the user that always includes:

```json
{
  "dates": ["YYYY-MM-DD"],
  "entries": [
    {
      "date": "YYYY-MM-DD",
      "taskId": 123456,
      "taskTitle": "Issue subject/title",
      "hours": 2,
      "activityId": 5,
      "activity": "Development",
      "comment": "Development work"
    }
  ],
  "totalHours": 2
}
```

`taskId`, `taskTitle`, and `activity`/`activityId` are mandatory in the preview output for each entry.

| Step | Field | Question `id` | Submit |
|------|--------|---------------|--------|
| 1 | `dates` | — | Today / Multiple / Other or ISO dates |
| 2 | `dateIssues` | `YYYY-MM-DD` | `{ "2026-05-25": [169305, 168545], ... }` — multi-select per date; comma string if Other |
| 3 | `pairHours` | `YYYY-MM-DD:issueId` | `{ "2026-05-25:169305": 2, ... }` |
| 4 | `pairActivities` | `YYYY-MM-DD:issueId` | First option `default — Development` (or env default), then cached activities e.g. `11 — QA`. Pass `activityBatchId` from `data` with answers. |
| 5 | `pairComments` | `YYYY-MM-DD:issueId` | `{ "2026-05-25:169305": "default", ... }` |
| 6 | `confirmSave` | — | `"yes"` after preview |

There is **no** `pairIncludes` yes/no step — issues chosen in step 2 are what get logged.

## Write actions

- Always ask an explicit final **yes/no** confirmation after showing the preview schema.
- Save only when user answer is **yes** and pass **`confirmSave: "yes"`**.
- If user answer is **no**, do not save and end the flow without making any write call.
- For multiple dates/issues in one request, do **not** use team `log_time`; use `redmine_agent_log_time`.
