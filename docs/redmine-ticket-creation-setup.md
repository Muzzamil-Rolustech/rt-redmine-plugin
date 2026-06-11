# Redmine Ticket Creation Setup Guide

This guide explains how to set up Cursor for creating Redmine tickets from a Google Sheet using the RT Redmine Plugin.

## Prerequisites

Before starting, make sure you have:

- A valid Redmine API key.
- A Google service account JSON file.
- Access to the Google Sheet that will be used for ticket creation.
- Cursor installed on your machine.

## Install the Plugin

Install the RT Redmine Plugin from GitHub:

https://github.com/Muzzamil-Rolustech/rt-redmine-plugin

Install it at the **project level**. Project-level installation is recommended because Cursor may not always update account-level plugin versions reliably after plugin changes.

## Configure MCP

Update the plugin MCP configuration with your local values:

- `REDMINE_BASE_URL`
- `REDMINE_API_KEY`
- `REDMINE_ACTIVITY_ID`
- `REDMINE_BILLABLE_HOURS_FIELD_ID`
- `SERVICE_ACCOUNT_PATH`
- `DRIVE_FOLDER_ID`

Keep real API keys and service account paths out of Git.

## Share the Google Sheet

The Google Sheet used for ticket creation must be shared with the Google service account.

Grant the service account **Editor** access. This is required because the agent writes created Redmine ticket IDs back into the sheet after ticket creation.

The `Ticket` column is used for Redmine ticket IDs. If the sheet does not already have a `Ticket` column, the agent can create one and write the ticket IDs there.

## How to Ask the Agent

In Cursor chat, attach or paste the Google Sheet link and mention the sheet/tab name.

Example:

```text
Create Redmine tickets from this sheet:
https://docs.google.com/spreadsheets/d/<sheet-id>/edit

Sheet name: Sheet1
```

## Ticket Creation Workflow

1. The agent reads the user-provided Google Sheet.
2. Cursor may ask for MCP tool approval. You can allow the required calls, or add trusted calls to the allowlist for a smoother process.
3. The agent asks you to select the Redmine project.
4. The agent asks you to select the target version.
5. The agent shows the detected column mapping for confirmation.
6. The agent shows a final preview of the tickets that will be created.
7. After approval, the agent creates the tickets in Redmine.
8. The agent writes the created Redmine ticket IDs back to the sheet's `Ticket` column.

## Standard Sheet Reference

Use this sheet only as a reference for the expected format:

https://docs.google.com/spreadsheets/d/1AVS1EmSpwYNhB2eN3f4NoWivQxwkzvj9_MXLwQeWA3g/edit?gid=0#gid=0

For actual ticket creation, always provide the real sheet link in chat.
