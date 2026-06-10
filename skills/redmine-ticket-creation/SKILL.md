---
name: redmine-ticket-creation
description: Create Redmine issues end-to-end using local redmine_agent_create_issue workflow.
---

# Redmine ticket creation

## Tool routing

- **Single issue:** `redmine_agent_create_issue`.
- **Spreadsheet / many issues:** `redmine_agent_batch_create_issues` — agent maps columns via `columnMapping` (see `redmine-batch-ticket-creation` skill).
- **Trackers / statuses / priorities:** `redmine_agent_get_config` — live enumerations from Redmine (same cache as MCP connect). Use tracker names or `id — name` from config; aliases include `bug`→Bug, `cr`→Change Request, `story`→User Story.
- Do **not** use team `redmine` `create_issue` for full workflow creation.

## Batch Steps to Reproduce (Task + Bug)

When issues are created via **`redmine_agent_batch_create_issues`**, **Task** and **Bug** rows receive the **Steps to Reproduce** custom field (never blank):

| Priority | Source |
|----------|--------|
| 1 | **Comments / Question** column (when non-empty) |
| 2 | **Description** column (when no comments) |
| 3 | **Task / Sub-Task title** (issue subject) |
| 4 | Fallback `"..."` |

Issue description still merges Description + Comments; Steps to Reproduce uses **comments first**, then description, then subject.

**Bug rows:** Steps to Reproduce must be preset on **create** with the Bug tracker. If Redmine creates as Task instead (parent tracker rules), the row fails and is not written back to the sheet — see `redmine-batch-ticket-creation` skill for recovery (`existingNodeIds`, fix orphan issue).

Workflow resolves trackers from live config — not hard-coded ids.

## Required interaction contract

When tool response has:
- `needsInput=true`
- `data.batchMode=true`
- `questions[]`

You must:
- Render **all** questions in **one AskQuestion**.
- Avoid free-form `key=value` chat collection while `questions[]` are available.
- If the user says to stop/cancel current creation flow, stop immediately and do not continue with pending intake.

## Intake pattern in current workflow

Choice fields:
- `project`, `tracker`, `status`, `priority`

Text fields are paired as:
- `<field>Mode` with options:
  - `default`
  - `Other` (use this when user wants to enter text)
- `<field>Custom` text input (used when mode is `Other`)

Interpretation:
- mode `Other` + custom input present => use custom value
- mode `default` => keep default value
- for non-mandatory fields, no selection (and empty custom) => blank
- `subjectCustom` must be non-empty (subject required)
- Do not require explicit "Custom" choice label; use `Other`.
- Continue should work even if optional fields are left unselected.

## Completion flow

1. Gather full intake in one AskQuestion.
2. Retry `redmine_agent_create_issue` with all answers.
3. Show preview from `data.previewText` / `structuredIssue`.
4. Ask explicit confirmation.
5. Retry with `confirmSave: "yes"` to post.

## Error handling

- If custom field mapping is unavailable/forbidden, continue creation and surface warnings.
- Surface API validation errors exactly as returned.
- If confirmation is not yes, do not save.
