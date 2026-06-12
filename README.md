# RT Redmine Plugin

Cursor plugin bundle for Redmine operations:

- MCP server: `@muzzamil-khan/redmine-agent-mcp` (via `npx`)
- Google Sheets MCP server: `mcp-google-sheets@latest` (via `uvx`)
- Skills: `redmine-operations`, `redmine-ticket-creation`, `redmine-batch-ticket-creation`
- Rule: bootstrap rule to load the skills

## What this plugin provides

- Redmine MCP config for published `@muzzamil-khan/redmine-agent-mcp`
- Google Sheets MCP config for reading spreadsheet payloads and writing Redmine links back
- Reusable skills for Redmine operations, single ticket creation, and batch ticket creation
- A minimal rule that ensures the Redmine skills are loaded

## Quick start

1. Install the runtime tools below (`Node.js` + `uv`).
2. Install or update **Cursor** to the latest version (2.6+ recommended for plugin MCP support).
3. Install this plugin in Cursor (project-level install is recommended).
4. Follow the setup guide to configure credentials and MCP env vars:
   - [Redmine Ticket Creation Setup Guide](docs/redmine-ticket-creation-setup.md)

## Prerequisites

Both MCP servers run through Cursor. You need these tools on your machine:

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

### Project-level install (recommended)

Install at the **project level** when possible. Project-level installs pick up plugin updates more reliably than some account-level installs after plugin changes.

1. Open **Cursor Settings** → **Plugins** (or the marketplace panel).
2. Install **RT Redmine Plugin** from your team marketplace, or from the repo if your org has imported it.
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

## MCP servers in this plugin

Cursor starts these from `mcp.json`:

### Redmine agent (`npx`)

```bash
npx -y @muzzamil-khan/redmine-agent-mcp
```

Environment variables:

- `REDMINE_BASE_URL`
- `REDMINE_API_KEY`
- `REDMINE_ACTIVITY_ID`
- `REDMINE_BILLABLE_HOURS_FIELD_ID`

### Google Sheets (`uvx`)

```bash
uvx mcp-google-sheets@latest
```

Always use `@latest` so `uvx` fetches the newest `mcp-google-sheets` release.

Environment variables (OAuth flow used by this plugin):

- `CREDENTIALS_PATH` — absolute path to your Google OAuth client JSON
- `TOKEN_PATH` — writable path where the OAuth refresh token is cached after first login (any path you choose)

See [docs/redmine-ticket-creation-setup.md](docs/redmine-ticket-creation-setup.md) for Google Cloud setup, Redmine API key, sheet sharing, and first-run OAuth steps.

Keep real API keys and credential paths out of Git. The checked-in `mcp.json` uses placeholders.

### Enable MCP servers in Cursor

1. Open **Settings** (`Cmd+Shift+J` / `Ctrl+Shift+J`)
2. Go to **Features → Model Context Protocol**
3. Enable `redmine-agent` and `google-sheets`

## Included skills

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
