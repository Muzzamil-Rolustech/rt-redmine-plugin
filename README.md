# RT Redmine Plugin

Cursor plugin bundle for Redmine operations:

- MCP server: `@muzzamil-khan/redmine-agent-mcp`
- Google Sheets MCP server for spreadsheet import/write-back
- Skills: `redmine-operations`, `redmine-ticket-creation`, `redmine-batch-ticket-creation`
- Rule: bootstrap rule to load the skills

## What this plugin provides

- Redmine MCP config for published `@muzzamil-khan/redmine-agent-mcp`
- Google Sheets MCP config for reading spreadsheet payloads and writing Redmine links back
- Reusable skills for Redmine operations, single ticket creation, and batch ticket creation
- A minimal rule that ensures the Redmine skills are loaded

## MCP server commands

The Redmine agent uses the published npm package:

```bash
npx -y @muzzamil-khan/redmine-agent-mcp
```

The Google Sheets MCP uses:

```bash
uvx mcp-google-sheets@latest
```

## Environment variables

Set these in `mcp.json` before using the plugin:

- `REDMINE_BASE_URL`
- `REDMINE_API_KEY`
- `REDMINE_ACTIVITY_ID`
- `REDMINE_BILLABLE_HOURS_FIELD_ID`
- `SERVICE_ACCOUNT_PATH`
- `DRIVE_FOLDER_ID`

Keep real API keys and service account paths out of the repository. The checked-in `mcp.json` uses placeholders.

## Included Skills

- `redmine-operations`: Routes Redmine reads, updates, time logging, config lookup, permissions, and assignee lookup.
- `redmine-ticket-creation`: Creates a single Redmine issue through MCP intake, preview, and explicit confirmation.
- `redmine-batch-ticket-creation`: Creates Feature → User Stories → Tasks from Google Sheets or CSV with mapping, preview, confirmation, and sheet write-back.

## Standard Sheet Reference

Use this sheet as a reference for the expected batch ticket format:

https://docs.google.com/spreadsheets/d/1AVS1EmSpwYNhB2eN3f4NoWivQxwkzvj9_MXLwQeWA3g/edit?gid=0#gid=0

Users should provide the actual sheet link when asking the agent to create tickets. This reference URL documents the expected format only.

In batch creation, the `Ticket` column is the Redmine ticket id/write-back column. Leave it blank for new issues; after creation, the agent fills the user-provided sheet with `Ticket:#<taskId>` and/or `MainTicket:#<storyId>` links.

## Extend with more skills

Add new skills under:

`skills/<new-skill>/SKILL.md`

All Redmine skills can share the same `redmine-agent` MCP server from `mcp.json`. Spreadsheet flows also use the `google-sheets` MCP server.
