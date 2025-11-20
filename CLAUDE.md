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

## CRM Process Automation & Agents

### Process Architecture

This project implements a **process-to-agent mapping** where each daily CRM workflow is automated by a specialized AI agent/skill. See `plano-atualizado.md` for the complete strategic plan and implementation roadmap.

**Current Implementation Status:**

| Process | Agent/Skill | Status |
|---------|-------------|--------|
| 2.2 - Avanço da Oportunidade | Opportunity Advancer | ✅ Operational |
| 3.0 - Weekly Pipeline Review | Weekly Pipeline Digest | 🎯 Priority 1 |
| 2.1 - Preparação para Encontro | Meeting Prep Briefer | 📋 Planned (Phase 2) |
| 1.0 - Entrada de Oportunidades | Opportunity Creator | 📋 Planned (Phase 2) |
| 4.0 - Follow-up Management | Follow-up Drafter | 📋 Planned (Phase 3) |
| 5.0 - Orquestração Integrada | CRM Orchestrator | 🔮 Future (Phase 4) |

### Process 2.2: Opportunity Advancement (Operational)

**Trigger:** `"Atualize o avanço da oportunidade [Nome]"`

**Workflow:**
1. AI loads most recent meeting notes (Anotações) or Offline Notes
2. AI analyzes transcripts/notes and auto-fills 5 fields:
   - **Update Log**: 1 concise sentence (max 10 words)
   - **Next Action**: Clear objective action (verb + object)
   - **Next Action Date**: AI suggests date (last interaction + 7 days default)
   - **Biz Funnel**: Detects stage advancement signals
   - **Status**: In Progress / Waiting Feedback / Scheduled
3. User reviews and confirms changes
4. Pipeline moves forward with clear next action

**Documentation:** Fully documented in Notion at `/Processos de Negócio - CRM Workflow - 2.2 - Avanço`

**AI Configuration:** Implemented in My Notion AI with command syntax and processing logic

### Weekly Pipeline Digest (Priority 1)

**Purpose:** Automated weekly pipeline health report identifying risks, opportunities, and action items. Replaces 30 minutes of manual view reviews with 5 minutes of actionable digest.

**Output Includes:**
- Pipeline Health Score (velocity, conversion rate, stalled count)
- Immediate action items (overdue, stalled >14 days)
- Revenue opportunities (high-momentum deals)
- Weekly agenda (Next Actions by date)

**Status:** Specified in `plano-atualizado.md`, ready for implementation

### Future Vision: Orchestrated Workflows

Agents will eventually work together in integrated workflows. Example:
- **Trigger:** New meeting note created (Anotação)
- **Workflow:** Opportunity Advancer processes transcript → Detects stage advancement → Meeting Prep Briefer generates briefing for next meeting → Follow-up Drafter creates thank-you email
- **Result:** Pipeline advanced, next meeting prepped, follow-up drafted - all automated

**Timeline:** Phase 4 (weeks 11-16), after foundational agents are stable

### Key References

- **Strategic Plan:** `plano-atualizado.md` - Complete architecture, roadmap, and decisions
- **Process Documentation:** Notion `/Processos de Negócio - CRM Workflow/` pages
- **AI Configuration:** Notion `/My Notion AI` - Agent instructions and commands

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
