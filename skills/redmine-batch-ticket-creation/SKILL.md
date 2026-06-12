---
name: redmine-batch-ticket-creation
description: Batch-create Redmine Feature → User Stories → Tasks from a Google Sheet or CSV. Human-readable breakdown, yes/no confirmation, AskQuestion for wrong assignees — never upload without user approval.
---

# Redmine batch task creation (Google Sheet or CSV)

Use this skill for batch / spreadsheet / Google Sheet issue creation.

**Do not** read repo source files, grep the codebase, or invent column names. Use **MCP tool responses only** (`get_config`, `batch_create_issues` steps, `suggestedColumnMapping`, `priorityOptions`, `trackerOptions`).

---

## Fixed step order (no skipping)

| Step | Action |
|------|--------|
| **0** | Load sheet → `get_sheet_data` → `redmine_agent_sheet_to_csv` |
| **1** | **`redmine_agent_get_config`** (`force: true`) — trackers, priorities, statuses, aliases for mapping |
| **2** | `batch_create_issues` (no `project`) → **`pick_project`** → **AskQuestion** |
| **3** | Retry with `project` → **`pick_target_version`** → **AskQuestion always** (never infer from missing sheet column) |
| **4** | Retry with `missingTargetVersionFix` → **`map_columns`** → use **`data.suggestedColumnMapping`** |
| **5** | **`confirm_mapping`** → AskQuestion proceed / change → `confirmMappingAction: "proceed"` |
| **6** | **`fix_batch_gaps`** / **`fix_assignees`** if returned — AskQuestion using `priorityOptions` / `trackerOptions` / `statusOptions` / `assigneeOptions` from MCP |
| **7** | **`preview_gate`** → post `previewOverview` + `previewDiagram` + `previewTree` → AskQuestion yes/no |
| **8** | `previewAction: "proceed"` only after **yes** |
| **9** | Sheet write-back when `sheetSource` present — **all sub-steps mandatory, none skippable:** **9a** unmerge id column → **9b** Feature title HYPERLINK (when `writeBackHints.featureWriteBack`) → **9c** Ticket / MainTicket HYPERLINKs |

**Never** upload on the first `batch_create_issues` call. **Never** use shell/scripts to create issues.

---

## Step 0 — Load sheet

```json
get_sheet_data({ "spreadsheet_id": "<id>", "sheet": "Sheet1" })
```

```json
redmine_agent_sheet_to_csv({
  "sheetsPayload": "<get_sheet_data result>",
  "spreadsheetId": "<id>",
  "sheet": "Sheet1"
})
```

Keep **`data.csvContent`** and **`data.sheetSource`** for every batch call in the session.

---

## Step 1 — Redmine config (always first)

```json
redmine_agent_get_config({ "force": true })
```

Use for:

- **Trackers** + `trackerAliases` (Type column, e.g. `bug`→Bug, `cr`→Change Request)
- **Priorities** + `priorityAliases` (e.g. `medium`→Normal, `highest`→Immediate)
- **Statuses** + `statusAliases` (Status column — see table below)
- Preview labels (`id — name`)

Do **not** search the codebase for enum values — config is the source of truth.

### Status aliases (sheet → Redmine)

| Sheet label | Redmine status |
|-------------|----------------|
| To Do | New |
| In Progress | In Progress |
| Review | Code Review |
| On Hold | Waiting for Support |
| Dev Done | Ready For QA |
| Done | Closed |
| Feedback | Feedback |
| Deferred | Deferred |
| Deployed on Staging | Deployed on Staging |
| Deployed on PreProd / Deplyed on PreProd | Pre-production |
| Ready For Pre-Prod / Ready For PreProd | Move to preprod |
| Verified Pre-Prod / Verified PreProd | Ok for Release |

- Map sheet **Status** column via `suggestedColumnMapping.status` (header `Status`).
- **Empty** Status on a row → workflow default **New**.
- Unknown labels (no alias / no exact match) → **`fix_batch_gaps`** AskQuestion → retry with **`statusNameFixes`**.

---

## Step 2 — Project

```json
redmine_agent_batch_create_issues({
  "csvContent": "<from sheet_to_csv>",
  "sheetSource": { "spreadsheetId": "<id>", "sheetTitle": "Sheet1" }
})
```

**AskQuestion** from `data.projectOptions`. Retry with `"project": "<id — name>"`.

Never default a project. Never infer from FE/BE/System column.

---

## Step 3 — Target version (always ask)

After `project` is set, call again **without** `missingTargetVersionFix` → `pick_target_version`.

**Always AskQuestion** — even when the sheet has no Target Version column and even when the project has zero versions.

### AskQuestion options

MCP lists existing project versions (when any) and **Skip target version**. No separate “Create new version” step — user types the name via **Other**.

| User picks | Retry with |
|------------|------------|
| `1314 — Sprint name` | `missingTargetVersionFix`: that exact label |
| **Other** / typed name in follow-up comment | `missingTargetVersionFix`: `"Create: <name>"` (creates version in Redmine, then continues) |
| **Skip target version** | `missingTargetVersionFix`: `"Skip target version"` |

When the project has **no** versions, the same AskQuestion still runs — user can **Other** (type a name) or **Skip**.

After creation, MCP resolves to `"id — name"` and continues to `map_columns`. Use that resolved value in later batch calls.

```json
{ "missingTargetVersionFix": "Create: Sprint 42" }
```

Sheet **Target Version** column (when mapped) overrides per User Story row. Batch default from this step applies to stories **without** a per-row value.

Before `previewAction: "proceed"`, copy the same version to every Task under versioned stories via `rowFixes.{nodeId}.targetVersion` (parser does not cascade to sub-tasks).

---

## Step 4 — Column mapping (use MCP suggestion)

Retry with `project` + `missingTargetVersionFix` (no `columnMapping`) → `map_columns`.

MCP returns **`data.suggestedColumnMapping`** — use it as `columnMapping`:

```json
{
  "headerRowIndex": 1,
  "columns": {
    "userStory": "User Stories",
    "subtask": "Tasks",
    "description": "Description",
    "hours": "Estimate",
    "assignedTo": "Assignee",
    "status": "Status",
    "ticket": "Ticket"
  }
}
```

### Recognized header layouts (built into MCP)

| Role | Headers matched automatically |
|------|----------------------------|
| **userStory** | `User Stories`, `User Story`, `Story`, or exact `Task` (not Task ID, not Sub*) |
| **subtask** | `Tasks`, `Sub-Tasks`, `Sub Task`, `Subtask`, … |
| **hours** | `Hours`, `Estimate`, `ETA` |
| **ticket** / **redmineId** | `Ticket`, `Task ID`, … |

### Ticket column — User Story + Task ids in one cell

When **ticket** (not Task ID) is the id column:

| Row type | Cell format | Meaning |
|----------|-------------|---------|
| User Story row (story only) | `MainTicket:#id` or `=HYPERLINK(url,"MainTicket:#id")` | User Story already on Redmine |
| User Story + Task same row | `=HYPERLINK(url1,"Ticket:#id") & " " & HYPERLINK(url2,"MainTicket:#id")` | Task link first, User Story second |
| Task-only row (story above) | `=HYPERLINK(url,"Ticket:#id")` | Task already on Redmine |

- **MainTicket** → User Story · **Ticket** → Task (subtask)
- Plain `Ticket:#id` / `MainTicket:#id` (no formula) still parses on read
- Bare `#id` / `=HYPERLINK(url,"#id")` still works (legacy Task ID column)
- **Never** use `Ticket:=HYPERLINK(...)` — that is not valid Google Sheets syntax

If **`suggestedColumnMapping.ambiguousRoles`** is non-empty → **AskQuestion** (options = `detectedHeaders`). Otherwise post the suggestion table and retry with `columnMapping` → **`confirm_mapping`**.

**AskQuestion** proceed / change → on proceed: `confirmMappingAction: "proceed"`.

Steps to Reproduce is **not** a sheet column — map `comments` / `description` roles only (`data.stepsToReproduceRule`).

---

## Step 5 — Fix gaps (config + AskQuestion only)

### Priority / Type / Status mismatch (`fix_batch_gaps`)

MCP returns **`questions[]`** with `priorityOptions` / `trackerOptions` / `statusOptions` from config.

**One AskQuestion** for all questions. Retry with:

```json
{
  "priorityNameFixes": { "Highest": "1 — Immediate" },
  "trackerNameFixes": { "CR": "5 — Change Request" },
  "statusNameFixes": { "On Hold": "19 — Waiting for Support" }
}
```

Aliases in config apply automatically when no fix is needed (`highest`→Immediate, `on hold`→Waiting for Support, etc.). For unmapped labels, use **`priorityNameFixes`** / **`trackerNameFixes`** / **`statusNameFixes`** — do not edit CSV by hand unless the user asks.

### Assignees (`fix_assignees`)

One AskQuestion per distinct invalid name in `invalidAssigneeGroups`. Retry with **`assigneeNameFixes`** (`"Leave unassigned"` / `"Skip this row"` supported).

---

## Step 6 — Preview + confirm

Post in order:

1. `data.previewOverview`
2. `data.previewDiagram` (fenced `mermaid`)
3. `data.previewTree`

**AskQuestion** yes / no. Upload only on **yes**.

---

## Step 7 — Upload

```json
{
  "csvContent": "<same session>",
  "sheetSource": { ... },
  "project": "<chosen>",
  "missingTargetVersionFix": "<chosen or Skip target version>",
  "columnMapping": { ... },
  "confirmMappingAction": "proceed",
  "assigneeNameFixes": { ... },
  "priorityNameFixes": { ... },
  "statusNameFixes": { ... },
  "rowFixes": { "row-8-subtask": { "targetVersion": "1314 — Sprint" } },
  "previewAction": "proceed"
}
```

---

## Step 9 — Sheet write-back (Google Sheets MCP only)

After successful upload, read `data.created[]` + `data.writeBackHints`. **Complete every sub-step below in order — do not skip Feature write-back.**

Skip the whole step only when there is no `sheetSource` (CSV-only batch).

### Feature (optional on sheet — mandatory write-back when created)

The **Feature** is **not** a mapped column role. It is detected automatically from an optional **title row above the header**, e.g. `Feature: My feature name` (often merged across the sheet).

- When the sheet has this row → batch creates a Redmine **Feature** parent.
- When `data.writeBackHints.featureWriteBack` is present → **you must** write the Feature HYPERLINK in **9b** (same upload session). **Never** write only Ticket/MainTicket links and omit the Feature cell.

### Step 9a — Detect merges + unmerge id column

```json
get_sheet_data({
  "spreadsheet_id": "<id>",
  "sheet": "Sheet1",
  "include_grid_data": true
})
```

From the response:

- `sheets[].properties.sheetId` — numeric id (often matches URL `gid`)
- `sheets[].merges[]` — `{ startRowIndex, endRowIndex, startColumnIndex, endColumnIndex }` (0-based, end exclusive)
- `data.writeBackHints.idColumnIndex` — 0-based Ticket / Task ID column (usually `0` = column A)

**Unmerge only the id column over data rows** (first data row through last created row). Do **not** include the Feature title row merge unless it is wholly inside this range.

Example — Ticket column A, data rows 3–22 (0-based row indices 2–22), `sheetId` 0:

```json
batch_update({
  "spreadsheet_id": "<id>",
  "requests": [
    {
      "unmergeCells": {
        "range": {
          "sheetId": 0,
          "startColumnIndex": 0,
          "endColumnIndex": 1,
          "startRowIndex": 2,
          "endRowIndex": 22
        }
      }
    }
  ]
})
```

- Range must **fully contain** each merge to split; partial overlap fails.
- If `merges` has **no** entry overlapping the id column in the data range, skip `unmergeCells`.
- Other columns (e.g. merged User Stories) stay merged — only unmerge the id column band.

### Step 9b — Feature title HYPERLINK (**mandatory when `featureWriteBack` present**)

When `data.writeBackHints.featureWriteBack` exists, write **first** (before Ticket column links):

```json
batch_update_cells({
  "spreadsheet_id": "<id>",
  "sheet": "Sheet1",
  "ranges": {
    "<featureWriteBack.columnLetter><featureWriteBack.rowIndex>": [
      ["=HYPERLINK(\"<featureWriteBack.issueUrl>\",\"Feature: <title> #<issueId>\")"]
    ]
  }
})
```

Use values from `featureWriteBack` (`rowIndex`, `columnLetter`, `issueUrl`, `issueId`, `title`, `formulaTemplate`). The **entire display text** is one clickable link:

```
Feature: My feature name #12345
```

Preserve the existing `Feature: …` prefix from the sheet; append ` #<id>` inside the HYPERLINK label (not as plain text before `=`).

### Step 9c — Ticket / Task ID HYPERLINK formulas

Use **`batch_update_cells`**. Every cell must start with `=` — the label (`Ticket:` / `MainTicket:` / `Feature:`) goes **inside** the HYPERLINK display text, not before the formula.

#### Task ID column (simple)

```
=HYPERLINK("https://redmine.example.com/issues/12345","#12345")
```

#### Ticket column (composite — preferred layout)

| Row type | Formula (one cell) |
|----------|-------------------|
| Task only | `=HYPERLINK("<taskUrl>","Ticket:#<taskId>")` |
| User Story only | `=HYPERLINK("<storyUrl>","MainTicket:#<storyId>")` |
| User Story + Task same row | `=HYPERLINK("<taskUrl>","Ticket:#<taskId>") & " " & HYPERLINK("<storyUrl>","MainTicket:#<storyId>")` |

**Rules**

- **Order:** Step **9a** unmerge → Step **9b** Feature title cell (when `featureWriteBack`) → Step **9c** Ticket column `batch_update_cells`. Never write Ticket links and skip Feature when `featureWriteBack` is present.
- Ticket link always before MainTicket when both are in one cell.
- Use `data.writeBackHints.formulaTemplate` and `featureWriteBack.formulaTemplate` — do not invent `Ticket:=HYPERLINK(...)` (Sheets will show it as plain text).
- When updating one level on a row that already has the other, preserve the existing segment and rebuild the combined formula.
- Feature title row: `=HYPERLINK("<url>","Feature: title #<id>")` in the cell that contains `Feature: …` (see `featureWriteBack.columnLetter` + `rowIndex`).
- One **created** issue → one cell on its `rowIndex` in the id column (task rows get `Ticket:#id`; story on same row also gets `MainTicket:#id`).

---

## Sheet → Redmine hierarchy

| Sheet | Redmine |
|-------|---------|
| Optional `Feature: …` title row above header (auto-detected; not a mapped column) | **Feature** parent + write-back `Feature: title #id` HYPERLINK in that cell |
| **User Stories** / **Task** column | **User Story** |
| **Tasks** / **Sub-Tasks** column | **Task** (or Type override) |
| **Type** | Tracker from config |
| **Status** | Redmine status from config (default **New** when empty) |
| **Ticket** / **Task ID** | Existing id → skip create; write-back after upload (Ticket: Task, MainTicket: User Story) |

---

## MCP step reference

| `data.step` | Agent action |
|-------------|--------------|
| `pick_project` | AskQuestion → retry with `project` |
| `pick_target_version` | **Always** AskQuestion → retry with `missingTargetVersionFix` (select existing / Other → `Create: <name>` / Skip) |
| `map_columns` | Use `suggestedColumnMapping` → retry with `columnMapping` |
| `confirm_mapping` | AskQuestion → `confirmMappingAction: "proceed"` |
| `fix_batch_gaps` | AskQuestion from `questions[]` → `priorityNameFixes` / `trackerNameFixes` / `statusNameFixes` |
| `fix_assignees` | AskQuestion → `assigneeNameFixes` |
| `preview_gate` | Preview blocks → AskQuestion yes/no |
| upload | `previewAction: "proceed"` after yes |

---

## Forbidden

- Upload on first batch call
- Searching repo files for trackers/priorities/column rules (use `get_config` + MCP responses)
- Auto-guessing assignees
- Skipping target-version AskQuestion because the sheet lacks a Target Version column
- Asking column mapping when `suggestedColumnMapping` is unambiguous
- Shell / tsx upload bypass
- Skipping **Step 9b** Feature HYPERLINK when `data.writeBackHints.featureWriteBack` is present
- Writing Ticket/MainTicket links (9c) without Feature title link (9b) in the same session

---

## Tools

| Tool | Use |
|------|-----|
| `get_sheet_data` | Read spreadsheet |
| `redmine_agent_sheet_to_csv` | → `csvContent` + `sheetSource` |
| **`redmine_agent_get_config`** | **Step 1 — trackers, priorities, statuses, aliases** |
| `redmine_agent_batch_create_issues` | Parse, preview, upload |
| `redmine_agent_get_all_users` | Assignee options (`projectId`) |
| `get_sheet_data` (`include_grid_data: true`) | Step 9a — `sheetId` + `merges` for id column |
| `batch_update` | Step 9a — `unmergeCells` on id column data range |
| `batch_update_cells` | Step 9b Feature link + Step 9c Ticket/MainTicket links |
