# Actioner Marketplace

Plugin marketplace for [Claude Desktop](https://claude.ai) — customer intelligence tools powered by [Actioner](https://actioner.com).

## Overview

This repository hosts officially maintained Claude Desktop plugins that connect Claude to Actioner's unified customer data platform. Each plugin bundles MCP server configuration, slash commands, and AI skills into a self-contained package that can be installed directly from the Claude Desktop plugin directory.

## Available Plugins

| Plugin | Version | Description |
|--------|---------|-------------|
| [Actioner Core](plugins/core/) | 0.1.0 | Query deals, customers, contacts, tickets, health scores, and more through a unified data model |

## Getting Started

### Install from Claude Desktop

1. Open [Claude Desktop](https://claude.ai) and navigate to **Settings > Connectors**.
2. Search for **Actioner** in the plugin directory.
3. Click **Connect** and complete OAuth sign-in with your Actioner account.
4. Select your organization if you belong to multiple workspaces.

No API keys or environment variables are required.

### Install Manually

Clone this repository and point Claude Desktop at a plugin directory:

```bash
git clone https://github.com/actioner/actioner-marketplace.git
```

Then add the MCP server from the plugin's `.mcp.json` to your Claude Desktop configuration.

## Plugin Structure

Each plugin lives under `plugins/<name>/` and follows this layout:

```
plugins/<name>/
├── .claude-plugin/
│   └── plugin.json          # Plugin metadata (name, version, description, keywords)
├── .mcp.json                # MCP server connection configuration
├── README.md                # Plugin-specific documentation
├── commands/                # Slash commands available to Claude
│   └── <command>.md         # Command definition with allowed tools and instructions
└── skills/                  # AI skills and reference material
    └── <skill-name>/
        ├── SKILL.md          # Skill instructions and data model reference
        └── references/       # Supporting docs (schemas, query examples, etc.)
```

### Key Files

- **`plugin.json`** — Declares the plugin's identity, version, and keywords for marketplace discovery.
- **`.mcp.json`** — Defines the MCP server(s) the plugin connects to, including transport type and URL.
- **`commands/`** — Markdown files that define slash commands (e.g., `/briefing`, `/deal-pipeline`). Each specifies allowed tools and step-by-step instructions for Claude.
- **`skills/`** — Reference material that Claude loads before performing complex tasks like SQL query generation against the Actioner data model.

## Marketplace Registry

The [`marketplace.json`](marketplace.json) file at the repository root serves as the plugin registry. It contains metadata for all available plugins including:

- Plugin identity, version, and description
- MCP server configuration and authentication requirements
- Available commands, skills, and tools
- Supported data sources and integrations
- Links to documentation and privacy policies

Tooling and CI pipelines can consume this file to validate, index, and publish plugins.

## Actioner Core Plugin

The flagship plugin connects Claude to Actioner's customer intelligence platform, aggregating data from:

- **CRM** — HubSpot, Salesforce
- **Support** — Zendesk, Freshdesk
- **Email** — Gmail, Outlook
- **Meeting transcription** — Gong, Fathom, TLDV, Fireflies

### Commands

| Command | Description |
|---------|-------------|
| `/briefing` | Daily briefing — urgent tickets, at-risk deals, upcoming closes, health alerts |
| `/search-customers` | Search customers, contacts, or companies by name, industry, or criteria |
| `/deal-pipeline` | View and analyze the deal pipeline by stage, rep, time period, or health |
| `/customer-health` | Check customer health scores, risk factors, and stakeholder sentiment |

### Tools

**Read-only:** Search Entities, Query Data, Get Deal, Get Company, Get Contact, Get Ticket, Get Task, Get Conversation, Daily Briefing, Meeting Prep, List Documents, Get Document Content

**Add:** Add Deal, Add Task, Add Person, Add Company, Add Entitlement, Add Note

**Update:** Update Deal, Update Company, Update Person, Update Task, Update Entitlement, Update Note

**Communication:** Compose Email, Create Meeting, Update Meeting

For full details, see the [Actioner Core README](plugins/core/README.md).

## Contributing

To add a new plugin to the marketplace:

1. Create a new directory under `plugins/<your-plugin-name>/`.
2. Add the required files: `.claude-plugin/plugin.json`, `.mcp.json`, `README.md`.
3. Add commands under `commands/` and skills under `skills/` as needed.
4. Register the plugin in `marketplace.json` at the repository root.
5. Open a pull request with your changes.

## Support

- **Email:** hello@actioner.com
- **Website:** [actioner.com](https://actioner.com)
- **Documentation:** [docs.actioner.com](https://docs.actioner.com)

## License

MIT
