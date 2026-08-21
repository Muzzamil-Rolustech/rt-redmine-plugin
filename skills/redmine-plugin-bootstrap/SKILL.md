---
name: redmine-plugin-bootstrap
description: Bootstrap guidance for the RT Redmine plugin. Use whenever the user mentions Redmine, tickets, issues, time logging, or Google Sheets ticket workflows so the correct Redmine skills are applied.
---

# Redmine plugin bootstrap

Use Redmine skills for tool-specific behavior:

- `skills/redmine-operations/SKILL.md`
- `skills/redmine-ticket-creation/SKILL.md`
- `skills/redmine-batch-ticket-creation/SKILL.md`

Batch create flow: `get_sheet_data` → `redmine_agent_sheet_to_csv` → `redmine_agent_get_config` → `redmine_agent_batch_create_issues`. Follow the `redmine-batch-ticket-creation` skill: project → target version (always ask) → use `suggestedColumnMapping` → confirm → preview → upload → sheet write-back with Google Sheets MCP.

Keep routing in those skills; do not invent alternate MCP tool sequences.
