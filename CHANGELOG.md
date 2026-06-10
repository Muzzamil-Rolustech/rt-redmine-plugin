# Changelog

## 1.1.0

- Added Google Sheets MCP config for spreadsheet import and Redmine link write-back.
- Added `redmine-ticket-creation` skill for single issue creation with preview and explicit save confirmation.
- Added `redmine-batch-ticket-creation` skill for Feature → User Stories → Tasks creation from Google Sheets/CSV.
- Updated `redmine-operations` skill for config lookup, assignee lookup, permission checks, issue creation, batch ticket creation, and activity-aware time logging.
- Updated plugin bootstrap rule to load all Redmine skills and enforce the batch creation flow.
- Updated plugin metadata and documentation for the published `@muzzamil-khan/redmine-agent-mcp` package.

## 1.0.1

- Initial plugin scaffold.
- Added plugin manifest.
- Added Redmine MCP config.
- Added `redmine-operations` skill.
- Added bootstrap rule and documentation.