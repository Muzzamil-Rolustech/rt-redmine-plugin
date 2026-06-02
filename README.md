# RT Redmine Plugin

Cursor plugin bundle for Redmine operations:

- MCP server: `@muzzamil-khan/redmine-agent-mcp`
- Skill: `redmine-operations`
- Rule: bootstrap rule to load the skill

## What this plugin provides

- One MCP server config for Redmine operations
- A reusable Redmine skill that can be extended with more skills later
- A minimal rule that ensures the skill is loaded

## MCP server command

The plugin uses this command:

```bash
npx -y @muzzamil-khan/redmine-agent-mcp
```

## Environment variables

Set these in the MCP config:

- `REDMINE_BASE_URL`
- `REDMINE_API_KEY`
- `REDMINE_ACTIVITY_ID`
- `REDMINE_BILLABLE_HOURS_FIELD_ID`

## Extend with more skills

Add new skills under:

`skills/<new-skill>/SKILL.md`

All skills can share the same `redmine-agent` MCP server from `mcp.json`.
