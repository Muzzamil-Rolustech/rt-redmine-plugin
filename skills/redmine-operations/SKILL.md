---
name: redmine-operations
description: Run Redmine operations via MCP (issues, projects, time entries, batch log time). Use when the user mentions Redmine, tickets, issues, projects, operational logs, or logging time.
disable-model-invocation: true
---

# Redmine Operations

Use the `redmine-agent` MCP server for Redmine actions.

## Scope

- Batch log time with `redmine_agent_log_time`
- Work from user intent in small guided steps
- Keep date and issue selections explicit in responses

## Workflow

1. If date is unclear, ask for date(s) first.
2. Ask for issue selection per date.
3. Ask for hours and comments for each selected issue/date pair.
4. Preview entries before save.
5. Save only after explicit user confirmation.

## Guardrails

- Never guess issue IDs or hours.
- Never save entries without confirmation.
- Keep all prompts concise and action-oriented.
