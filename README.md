# RT Redmine Plugin

Cursor + Claude Code plugin bundle for Redmine operations:

- MCP server: `@muzzamil-khan/redmine-agent-mcp` (via `npx`)
- Google Sheets MCP server: `mcp-google-sheets@latest` (via `uvx`)
- Skills: `redmine-operations`, `redmine-ticket-creation`, `redmine-batch-ticket-creation`, `redmine-plugin-bootstrap`
- Cursor rule: bootstrap rule to load the skills
- Claude packaging: `.claude-plugin/` + `.mcp.json` with `userConfig` defaults

## What this plugin provides

- Redmine MCP config for published `@muzzamil-khan/redmine-agent-mcp`
- Google Sheets MCP config for reading spreadsheet payloads and writing Redmine links back
- Reusable skills for Redmine operations, single ticket creation, and batch ticket creation
- Cursor Team Marketplace + Claude Code marketplace manifests
- Shared defaults for activity ID (`5`), billable field (`1`), SSE gateway URL/key, and base URL

## Quick start

1. Install the runtime tools below (`Node.js` + `uv`).
2. Install in **Cursor** and/or **Claude Code** (see install sections below).
3. Configure credentials (Cursor: **Plugins → Configure**; Claude: enable-time `userConfig` prompts).
4. Follow the setup guide:
   - [Redmine Ticket Creation Setup Guide](docs/redmine-ticket-creation-setup.md)

## Prerequisites

MCP servers run through Cursor or Claude Code. You need these tools on your machine:

| Tool | Used by | Purpose |
| --- | --- | --- |
| [Node.js](https://nodejs.org/en/download) (LTS) | `npx` | Runs `@muzzamil-khan/redmine-agent-mcp` |
| [uv](https://docs.astral.sh/uv/getting-started/installation/) | `uvx` | Runs `mcp-google-sheets@latest` |

### Install Node.js (Linux and Windows)

Download and install the current LTS release:

- https://nodejs.org/en/download

Verify:

```bash
node -v
npx -v
```

### Install uv (Linux)

Official installer script:

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```

Installer script: https://astral.sh/uv/install.sh

After install, restart your terminal (or source your shell profile) and verify:

```bash
uv --version
uvx --version
```

Other Linux options: [Homebrew](https://docs.astral.sh/uv/getting-started/installation/#homebrew), [pip/pipx](https://docs.astral.sh/uv/getting-started/installation/#pypi).

### Install uv (Windows)

Official PowerShell installer:

```powershell
powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex"
```

Installer script: https://astral.sh/uv/install.ps1

Restart the terminal, then verify:

```powershell
uv --version
uvx --version
```

Other Windows options: [WinGet](https://docs.astral.sh/uv/getting-started/installation/#winget), [Scoop](https://docs.astral.sh/uv/getting-started/installation/#scoop).

### Smoke-test the MCP commands

These are the same commands Cursor runs from `mcp.json`:

```bash
npx -y @muzzamil-khan/redmine-agent-mcp
```

```bash
uvx mcp-google-sheets@latest
```

If either command is not found, fix your `PATH` before configuring Cursor.

## Install the plugin in Cursor

Repository: https://github.com/Muzzamil-Rolustech/rt-redmine-plugin

### Use the latest Cursor version

Update Cursor before installing the plugin:

- **Help → Check for Updates** (or download from https://cursor.com)
- Use **Cursor 2.6+** so plugin MCP servers load correctly

### Team Marketplace (recommended)

This repo ships `.cursor-plugin/marketplace.json` so an admin can import it as a Team Marketplace:

1. Open **Dashboard → Plugins → Team Marketplaces**.
2. Click **Add Marketplace** / **Import from Repo**.
3. Paste `https://github.com/Muzzamil-Rolustech/rt-redmine-plugin`.
4. Confirm Cursor lists **rt-redmine-plugin**, then save access + install mode.
5. In Cursor, open **Customize**, find **RT Redmine Plugin**, and install (prefer **project scope**).
6. Open **Plugins → Configure** and set the required variables (`REDMINE_BASE_URL`, `REDMINE_API_KEY`, SSE URL/key, optional Sheets paths).

Marketplace metadata lives in:

- `.cursor-plugin/marketplace.json` — catalog entry (`source: "."`)
- `.cursor-plugin/plugin.json` — plugin manifest + `variables` schema
- `assets/logo.svg` — marketplace logo

### Project-level install

Install at the **project level** when possible. Project-level installs pick up plugin updates more reliably than some account-level installs after plugin changes.

1. Open **Customize** (or **Settings → Plugins**).
2. Install **RT Redmine Plugin** from your team marketplace.
3. Prefer **project scope** over user scope when prompted.

### Local development / manual install

For local testing, copy the plugin into Cursor's local plugins folder:

**Linux / macOS**

```bash
mkdir -p ~/.cursor/plugins/local/rt-redmine-plugin
cp -r /path/to/rt-redmine-plugin/. ~/.cursor/plugins/local/rt-redmine-plugin/
```

**Windows (PowerShell)**

```powershell
New-Item -ItemType Directory -Force -Path "$env:USERPROFILE\.cursor\plugins\local\rt-redmine-plugin"
Copy-Item -Recurse -Force "C:\path\to\rt-redmine-plugin\*" "$env:USERPROFILE\.cursor\plugins\local\rt-redmine-plugin\"
```

Then run **Developer: Reload Window** in Cursor.

## Install the plugin in Claude Code

This repo also ships Claude packaging:

- `.claude-plugin/marketplace.json` — marketplace catalog (`source: "./"`)
- `.claude-plugin/plugin.json` — plugin manifest + `userConfig` (with defaults)
- `.mcp.json` — MCP servers using `${user_config.*}` placeholders

### Add marketplace and install

```bash
claude plugin marketplace add Muzzamil-Rolustech/rt-redmine-plugin
claude plugin install rt-redmine-plugin@rt-redmine-marketplace
```

Or from a local checkout:

```bash
claude plugin marketplace add /path/to/rt-redmine-plugin
claude plugin install rt-redmine-plugin@rt-redmine-marketplace
```

On enable, Claude prompts for `userConfig` (API key required; activity/billable/SSE defaults are pre-filled).

### Local test without marketplace

```bash
claude --plugin-dir /path/to/rt-redmine-plugin
```

Validate packaging:

```bash
claude plugin validate /path/to/rt-redmine-plugin
```

## MCP servers in this plugin

- **Cursor** loads `mcp.json` with `${VAR}` placeholders (dashboard **Plugins → Configure**).
- **Claude Code** loads `.mcp.json` with `${user_config.*}` placeholders (`userConfig` on enable).

### Redmine agent (`npx`)

```bash
npx -y @muzzamil-khan/redmine-agent-mcp
```

Config values (Cursor variables / Claude `userConfig`):

| Variable | Default | Notes |
| --- | --- | --- |
| `REDMINE_BASE_URL` | `https://redmine.rolustech.com` | Override if needed |
| `REDMINE_API_KEY` | _(none — required)_ | Your personal Redmine API key |
| `REDMINE_ACTIVITY_ID` | `5` | Default time-entry activity |
| `REDMINE_BILLABLE_HOURS_FIELD_ID` | `1` | Billable hours custom field |
| `REDMINE_SSE_URL` | `https://staging5.rolustech.com:44312/sse` | Team SSE gateway |
| `REDMINE_SSE_API_KEY` | `rt-mcp-redmine` | Gateway `x-api-key` |

### Google Sheets (`uvx`)

```bash
uvx mcp-google-sheets@latest
```

Always use `@latest` so `uvx` fetches the newest `mcp-google-sheets` release.

Plugin variables / Claude `userConfig` (OAuth flow used by this plugin):

| Variable | Default | Notes |
| --- | --- | --- |
| `GOOGLE_SHEETS_CREDENTIALS_PATH` | `/path/to/client-secret.json` | Absolute path to OAuth client JSON (`CREDENTIALS_PATH`) |
| `GOOGLE_SHEETS_TOKEN_PATH` | `token.json` | Writable OAuth token cache path (`TOKEN_PATH`) |

See [docs/redmine-ticket-creation-setup.md](docs/redmine-ticket-creation-setup.md) for Google Cloud setup, Redmine API key, sheet sharing, and first-run OAuth steps.

Keep real API keys and credential paths out of Git. The checked-in `mcp.json` uses `${VAR}` placeholders only.

### Enable MCP servers in Cursor

1. Open **Settings** (`Cmd+Shift+J` / `Ctrl+Shift+J`)
2. Go to **Features → Model Context Protocol**
3. Enable `redmine-agent` and `google-sheets`

## Included skills

- `redmine-plugin-bootstrap`: Routes Redmine/Sheets requests to the correct skill (Claude + shared guidance; Cursor also has `rules/redmine-agent.mdc`).
- `redmine-operations`: Routes Redmine reads, updates, time logging, config lookup, permissions, and assignee lookup.
- `redmine-ticket-creation`: Creates a single Redmine issue through MCP intake, preview, and explicit confirmation.
- `redmine-batch-ticket-creation`: Creates Feature → User Stories → Tasks from Google Sheets or CSV with mapping, preview, confirmation, and sheet write-back.

## Standard sheet reference

Use this sheet as a reference for the expected batch ticket format:

https://docs.google.com/spreadsheets/d/1AVS1EmSpwYNhB2eN3f4NoWivQxwkzvj9_MXLwQeWA3g/edit?gid=0#gid=0

Users should provide the actual sheet link when asking the agent to create tickets. This reference URL documents the expected format only.

In batch creation, the `Ticket` column is the Redmine ticket id/write-back column. Leave it blank for new issues; after creation, the agent fills the user-provided sheet with `Ticket:#<taskId>` and/or `MainTicket:#<storyId>` links.

## Extend with more skills

Add new skills under:

`skills/<new-skill>/SKILL.md`

All Redmine skills can share the same `redmine-agent` MCP server from `mcp.json`. Spreadsheet flows also use the `google-sheets` MCP server.
