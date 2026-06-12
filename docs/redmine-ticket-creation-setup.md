# Redmine Ticket Creation Setup Guide

This guide walks you through configuring credentials, MCP servers, and Google Sheets so you can create Redmine tickets from a spreadsheet in Cursor.

For installing Node.js, `uv`, and the plugin itself, start with the [README](../README.md).

## Overview

| Piece | What it does |
| --- | --- |
| **RT Redmine Plugin** | Loads skills, rules, and MCP config into Cursor |
| **`redmine-agent` MCP** (`npx`) | Reads Redmine config, creates issues, writes ticket links |
| **`google-sheets` MCP** (`uvx`) | Reads your sheet and writes ticket IDs back after creation |

## Prerequisites checklist

Before you start, make sure you have:

- [ ] [Node.js](https://nodejs.org/en/download) installed (`npx` works)
- [ ] [uv](https://docs.astral.sh/uv/getting-started/installation/) installed (`uvx` works)
- [ ] **Latest Cursor** installed (3.6.3+ recommended) — **Help → Check for Updates**
- [ ] A valid **Redmine API key**
- [ ] A **Google Cloud OAuth client JSON** file (see below)
- [ ] **Editor** access to the Google Sheet used for ticket creation

## Step 1 — Install the plugin in Cursor

Repository: https://github.com/Muzzamil-Rolustech/rt-redmine-plugin

1. Open Cursor **Settings → Plugins** (or your team marketplace).
2. Install **RT Redmine Plugin**.
3. Prefer **project-level** installation when offered — it tends to pick up plugin updates more reliably.
4. Run **Developer: Reload Window** after install or upgrade.

If you are testing a local checkout, see the manual install steps in [README.md](../README.md#local-development--manual-install).

## Step 2 — Get your Redmine API key

1. Log in to your Redmine instance.
2. Open **My account** (or your profile).
3. Copy your **API access key** (or create one if needed).
4. Note your Redmine base URL, for example `https://redmine.example.com`.

You will set these in `mcp.json` as `REDMINE_BASE_URL` and `REDMINE_API_KEY`.

Also confirm with your Redmine admin:

- `REDMINE_ACTIVITY_ID` — default activity for time logging
- `REDMINE_BILLABLE_HOURS_FIELD_ID` — custom field id for billable hours (if used)

## Step 3 — Create Google OAuth credentials

This plugin's `mcp.json` uses the **OAuth 2.0 Desktop app** flow via `CREDENTIALS_PATH` and `TOKEN_PATH`.

### 3.1 Google Cloud project and APIs

1. Open [Google Cloud Console](https://console.cloud.google.com/).
2. Create or select a project.
3. Go to **APIs & Services → Library** and enable:
   - **Google Sheets API**
   - **Google Drive API**

### 3.2 OAuth consent screen

1. Go to **APIs & Services → OAuth consent screen**.
2. Choose **External** (or **Internal** if your org uses Google Workspace and that fits your policy).
3. Fill in the required app name and support email.
4. Add scopes:
   - `https://www.googleapis.com/auth/spreadsheets`
   - `https://www.googleapis.com/auth/drive`
5. Add your Google account as a **test user** while the app is in testing mode.

### 3.3 Create OAuth client ID and download JSON

1. Go to **APIs & Services → Credentials**.
2. Click **+ CREATE CREDENTIALS → OAuth client ID**.
3. Application type: **Desktop app**.
4. Name it (for example `cursor-redmine-sheets`).
5. Click **CREATE**, then **Download JSON**.
6. Save the file somewhere outside the repo, for example:
   - Linux: `~/.config/rt-redmine/google-oauth-client.json`
   - Windows: `C:\Users\<you>\.config\rt-redmine\google-oauth-client.json`

This downloaded file is your **`CREDENTIALS_PATH`** value.

> **Note:** This is Google OAuth, not Auth0. You are creating a Google OAuth Desktop client so `mcp-google-sheets` can access Sheets on your behalf.

### 3.4 Choose `TOKEN_PATH`

`TOKEN_PATH` is where the MCP server stores your OAuth refresh token after the first browser login.

- It can be **any writable file path** you choose.
- The directory must exist and be writable by your user.
- Keep it **outside Git** and **private** (it is as sensitive as a password).

Examples:

| OS | Example `TOKEN_PATH` |
| --- | --- |
| Linux | `/home/<you>/.config/rt-redmine/google-token.json` |
| Windows | `C:\Users\<you>\.config\rt-redmine\google-token.json` |
| macOS | `/Users/<you>/.config/rt-redmine/google-token.json` |

On first successful Google Sheets MCP use, Cursor may open a browser window so you can sign in and approve access. After that, the token file is reused.

### Alternative: service account (optional)

If you prefer a non-interactive service account instead of OAuth, use `SERVICE_ACCOUNT_PATH` in `mcp.json` and share each spreadsheet with the service account email (`client_email` in the JSON key). See the [mcp-google-sheets authentication docs](https://github.com/xing5/mcp-google-sheets#-authentication--environment-variables-detailed). The shipped plugin config uses OAuth (`CREDENTIALS_PATH` + `TOKEN_PATH`).

## Step 4 — Configure MCP in the plugin

Edit the plugin's `mcp.json` with your local values. Do **not** commit real secrets.

### Redmine (`redmine-agent`)

| Variable | Example | Notes |
| --- | --- | --- |
| `REDMINE_BASE_URL` | `https://redmine.example.com` | No trailing slash |
| `REDMINE_API_KEY` | `your_api_key` | From Redmine profile |
| `REDMINE_ACTIVITY_ID` | `5` | Your default activity id |
| `REDMINE_BILLABLE_HOURS_FIELD_ID` | `1` | Custom field id, if used |

### Google Sheets (`google-sheets`)

| Variable | Example | Notes |
| --- | --- | --- |
| `CREDENTIALS_PATH` | `/home/you/.config/rt-redmine/google-oauth-client.json` | Absolute path to OAuth client JSON |
| `TOKEN_PATH` | `/home/you/.config/rt-redmine/google-token.json` | Any writable path; created on first OAuth login |

Example snippet:

```json
{
  "mcpServers": {
    "redmine-agent": {
      "command": "npx",
      "args": ["-y", "@muzzamil-khan/redmine-agent-mcp"],
      "env": {
        "REDMINE_BASE_URL": "https://redmine.example.com",
        "REDMINE_API_KEY": "YOUR_API_KEY",
        "REDMINE_ACTIVITY_ID": "5",
        "REDMINE_BILLABLE_HOURS_FIELD_ID": "1"
      }
    },
    "google-sheets": {
      "command": "uvx",
      "args": ["mcp-google-sheets@latest"],
      "env": {
        "CREDENTIALS_PATH": "/absolute/path/to/google-oauth-client.json",
        "TOKEN_PATH": "/absolute/path/to/google-token.json"
      }
    }
  }
}
```

### Enable the servers in Cursor

1. **Settings → Features → Model Context Protocol**
2. Turn on **redmine-agent** and **google-sheets**
3. Reload the window if servers do not appear

### Verify MCP tools

In Cursor chat, confirm both servers are connected. The first Google Sheets call may trigger OAuth in the browser. Approve access with the same Google account that can edit your target sheet.

## Step 5 — Prepare the Google Sheet

The sheet used for ticket creation must be editable by the Google account you used for OAuth.

1. Open the spreadsheet in Google Sheets.
2. Confirm your OAuth Google account has **Editor** access.
3. Use or add a **`Ticket`** column for Redmine ticket IDs. If it is missing, the agent can create one and write ids there after creation.

Standard format reference (do not create tickets on this template unless intended):

https://docs.google.com/spreadsheets/d/1AVS1EmSpwYNhB2eN3f4NoWivQxwkzvj9_MXLwQeWA3g/edit?gid=0#gid=0

For real runs, always paste **your** sheet link in chat.

## Step 6 — Create tickets in Cursor

In Cursor chat, attach or paste the Google Sheet link and name the tab.

Example:

```text
Create Redmine tickets from this sheet:
https://docs.google.com/spreadsheets/d/<sheet-id>/edit

Sheet name: Sheet1
```

You can also invoke skills directly:

- `/redmine-batch-ticket-creation` — batch Feature → User Stories → Tasks from a sheet
- `/redmine-ticket-creation` — single issue with preview and confirmation
- `/redmine-operations` — reads, updates, time logging, config lookup

## Ticket creation workflow

1. The agent reads the Google Sheet via the `google-sheets` MCP.
2. Cursor may ask for MCP tool approval. Allow the calls, or add trusted tools to your allowlist.
3. The agent asks you to select the **Redmine project**.
4. The agent asks you to select the **target version**.
5. The agent shows detected **column mapping** for confirmation.
6. The agent shows a **final preview** of tickets to be created.
7. After you approve, tickets are created in Redmine via `redmine-agent`.
8. Created ticket IDs are written back to the sheet's **`Ticket`** column.

## Troubleshooting

| Problem | What to try |
| --- | --- |
| `spawn npx ENOENT` | Install Node.js and restart Cursor. Verify `npx -v` in a terminal. |
| `spawn uvx ENOENT` | Install `uv`, restart terminal and Cursor. On macOS/Linux, you may need the full path to `uvx` (for example `~/.local/bin/uvx`) in `mcp.json`. |
| Google OAuth fails | Check APIs are enabled, consent screen has test users, and `CREDENTIALS_PATH` is an absolute path. |
| Token errors | Delete `TOKEN_PATH` and sign in again on the next MCP call. |
| Sheet read/write denied | Sign in with a Google account that has **Editor** on that spreadsheet. |
| Plugin MCP not loading | Update Cursor to the latest version and run **Developer: Reload Window**. |
| Stale Google Sheets MCP | Keep `mcp-google-sheets@latest` in `args` so `uvx` pulls the newest release. |

## Related links

- [Plugin README](../README.md) — Node.js / `uv` install, `npx` / `uvx` commands, plugin install
- [mcp-google-sheets](https://github.com/xing5/mcp-google-sheets) — upstream Google Sheets MCP docs
- [Cursor Plugins docs](https://cursor.com/docs/plugins) — marketplace and local plugin testing
