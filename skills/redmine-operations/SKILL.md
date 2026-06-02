---
name: redmine-operations
description: Run Redmine operations via MCP (issues, projects, time entries, batch log time). Use when the user mentions Redmine, tickets, issues, projects, operational logs, or logging time.
---

# Redmine operations

Use **both** MCP servers from `.cursor/mcp.json`:

- **`redmine`**: team Westeros SSE gateway for standard Redmine operations.
- **`redmine-agent`**: local `npm run mcp` server with **only** `redmine_agent_log_time`.

| Server | Tool | Use for |
|--------|------|---------|
| **`redmine`** | Team SSE tools | Issues, projects, reads, updates |
| **`redmine-agent`** | `redmine_agent_log_time` | Batch log time + Billable Hours |

## Tool routing

| User intent | Tool |
|----|---|
| Log time / batch timelog / multiple issues or dates | **`redmine_agent_log_time`** |
| Get my time entries / logs | `get_my_time_entries` (team `redmine`) |
| My issues | `get_my_issues` (team `redmine`) |
| Get/search/update/create issues, projects, wiki | team `redmine` tools |
| Single-issue log (no batch) | Prefer **`redmine_agent_log_time`**; team `log_time` only if user insists |
| Everything else | team `redmine` |

## Required input discipline

- Use AskQuestion when a tool response has `needsInput` plus selectable options.
- When `questions[]` exists and `data.batchMode` is `true`, ask **all** questions in **one** AskQuestion.
- Never assume issue ID, hours, dates, or comment text unless explicitly provided by the user.

## Batch log time — one screen per step

When `data.batchMode` is true, show **all** `questions[]` in **one** AskQuestion.

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
      "comment": "Development work"
    }
  ],
  "totalHours": 2
}
```

`taskId` and `taskTitle` are mandatory in the preview output for each entry.

| Step | Field | Question `id` | Submit |
|------|--------|---------------|--------|
| 1 | `dates` | — | Today / Multiple / Other or ISO dates |
| 2 | `dateIssues` | `YYYY-MM-DD` | `{ "2026-05-25": [169305, 168545], ... }` — multi-select per date; comma string if Other |
| 3 | `pairHours` | `YYYY-MM-DD:issueId` | `{ "2026-05-25:169305": 2, ... }` |
| 4 | `pairComments` | `YYYY-MM-DD:issueId` | `{ "2026-05-25:169305": "default", ... }` |
| 5 | `confirmSave` | — | `"yes"` after preview |

There is **no** `pairIncludes` yes/no step — issues chosen in step 2 are what get logged.

## Write actions

- Always ask an explicit final **yes/no** confirmation after showing the preview schema.
- Save only when user answer is **yes** and pass **`confirmSave: "yes"`**.
- If user answer is **no**, do not save and end the flow without making any write call.
- For multiple dates/issues in one request, do **not** use team `log_time`; use `redmine_agent_log_time`.