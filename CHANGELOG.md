# Changelog

## 1.4.0

- Added Claude Code packaging: `.claude-plugin/plugin.json`, `.claude-plugin/marketplace.json`, and `.mcp.json`.
- Declared Claude `userConfig` with the same team defaults as Cursor variables (activity `5`, billable field `1`, SSE URL/key, base URL, Sheets paths).
- Added `skills/redmine-plugin-bootstrap` so Claude loads the same bootstrap guidance as the Cursor rule.

## 1.3.0

- Added `.cursor-plugin/marketplace.json` for Cursor Team Marketplace import (`source: "."`).
- Added `assets/logo.svg` and referenced it from the plugin + marketplace manifests.
- Cleaned `.cursor-plugin/plugin.json` for marketplace schema (moved display/category/tags to marketplace entry).
- Declared plugin `variables` and switched `mcp.json` to `${VAR}` placeholders for dashboard configuration.
- Preserved team defaults on variables (`REDMINE_ACTIVITY_ID=5`, `REDMINE_BILLABLE_HOURS_FIELD_ID=1`, SSE URL/key, base URL, Sheets paths).
- Fixed homepage/repository URLs to point at `rt-redmine-plugin`.

## 1.2.1

- Updated `mcp.json` configuration
    
## 1.2.0

- Synced skills, bootstrap rule, and `mcp.json` from `redmine-mcp-agent` workspace.
- Updated `redmine-operations` with bulk update routing (`redmine_agent_bulk_update_issues`).
- Updated `redmine-batch-ticket-creation` with status column mapping, status aliases, and target-version AskQuestion flow.
- Aligned Google Sheets MCP env vars with workspace config (`CREDENTIALS_PATH`, `TOKEN_PATH`).

## 1.1.0

- Added Google Sheets MCP config for spreadsheet import and Redmine link write-back.
- Added `redmine-ticket-creation` skill for single issue creation with preview and explicit save confirmation.
- Added `redmine-batch-ticket-creation` skill for Feature → User Stories → Tasks creation from Google Sheets/CSV.
- Updated `redmine-operations` skill for config lookup, assignee lookup, permission checks, issue creation, batch ticket creation, and activity-aware time logging.
- Updated plugin bootstrap rule to load all Redmine skills and enforce the batch creation flow.
- Updated plugin metadata and documentation for the published `@muzzamil-khan/redmine-agent-mcp` package.
- Added standard batch sheet documentation with the template URL and `Ticket` column write-back rules.

## 1.0.1

- Initial plugin scaffold.
- Added plugin manifest.
- Added Redmine MCP config.
- Added `redmine-operations` skill.
- Added bootstrap rule and documentation.