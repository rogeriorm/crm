# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a CRM project that leverages Claude Code with the Model Context Protocol (MCP) to integrate with Notion. The primary purpose is to enable AI-assisted management of business activities, opportunities, and data stored in Notion workspaces.

## Architecture

### MCP Integration
The project uses the Notion MCP server (`@notionhq/mcp-server-notion`) to provide Claude Code with direct access to Notion data through the MCP protocol. The server is configured in `.mcp.json` and automatically loads when the project is opened.

### Environment Configuration
- **Environment Variables**: Stored in `.env` (gitignored)
- **API Authentication**: Uses `NOTION_API_KEY` for Notion workspace access
- **Cross-Platform**: Configuration supports macOS, Windows, and Linux

### Permissions Model
Claude Code has pre-approved permissions for:
- `mcp__notion__notion-search` - Search Notion workspace without asking
- `mcp__notion__notion-fetch` - Fetch Notion pages/databases without asking

All other Notion operations (create, update, delete) require user approval.

## Setup

### Initial Configuration

```bash
# Copy environment template
cp .env.example .env

# Add your Notion API key to .env
# NOTION_API_KEY=secret_xxxxxxxxxxxxxxxxxxxx
```

### Verify Setup

```bash
# Check Node.js installation
node --version

# Test MCP server manually (optional)
npx -y @notionhq/mcp-server-notion
```

## Common Workflows

### Working with Notion Data

**Searching for business activities:**
- Use `mcp__notion__notion-search` with semantic queries
- Apply filters by date range (`created_date_range`) or creator (`created_by_user_ids`)
- Search within specific teamspaces using `teamspace_id`

**Fetching complete records:**
- Use `mcp__notion__notion-fetch` with page/database URL or ID
- Returns full content in Notion-flavored Markdown format
- Includes all properties, metadata, and nested content

**Creating/updating content:**
- Requires user approval (not in auto-approve list)
- Use appropriate tools: `notion-create-pages`, `notion-update-page`, etc.

### Data Model Understanding

The Notion workspace follows this structure:
- **Opportunities (Oportunidades)**: Business leads and deals with funnel stages
- **Notes (Anotações)**: Meeting notes and transcripts
- **Contacts (Contatos)**: People and relationships
- **Companies (Cliente)**: Client organizations
- **Tasks**: Action items linked to opportunities/notes

Key properties to understand:
- `Biz Funnel`: Tracks sales pipeline stages (Suspect, Credibilidade, Proposta, Oferta, etc.)
- `Priority`: High/Medium/Low urgency
- `Status`: Workflow state (In Progress, Waiting Feedback, Scheduled, etc.)
- Date properties: Use `date:{property}:start`, `date:{property}:end`, `date:{property}:is_datetime`

## Troubleshooting

### MCP Connection Issues
1. Verify `.env` file exists with valid `NOTION_API_KEY`
2. Restart Claude Code to reload MCP configuration
3. Check Node.js is in PATH: `which node` (macOS/Linux) or `where node` (Windows)

### Notion API Errors
- Ensure API key has access to the specific Notion workspace
- Verify integration has been shared with relevant pages/databases
- Check rate limits if receiving 429 errors

## Important Constraints

- **No source code**: This is a configuration-only project (no .py/.js/.ts files)
- **Data privacy**: Never commit `.env` or share actual API keys
- **Permission model**: Respect the auto-approve list; other operations require user consent
- **Cross-platform**: Configuration must remain compatible with macOS, Windows, and Linux
