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

### Approach: Pragmatic & Incremental

**Focus:** Deliver short-term improvements and validate technical approach before scaling.

**Strategy:** Hybrid model where agents exist in two layers:
- **Notion AI** (Layer 1): Native integration, configured in My Notion AI
- **Claude Code** (Layer 2): Programmatic mirrors with versioning, testing, logging

**Current Priority:** Validate hybrid approach by creating Claude Code mirror of existing Notion AI agent (Process 2.2), then build new agents incrementally.

### Implementation Status

| Phase | Agent | Status | Value |
|-------|-------|--------|-------|
| **Phase 0** 🎯 | **Process 2.2 Mirror** | 🚧 In Spec | Validate hybrid approach (2-3h) |
| Phase 0 | Process 2.2 (Notion AI) | ✅ Operational | Pipeline always moving |
| Phase 1 | Weekly Pipeline Digest | 📋 Specified | 30 min/week saved |
| Phase 2+ | Meeting Prep, Opp Creator, Follow-up | 💭 Planned | 80 min/week saved |

**Note:** Autonomous orchestration is long-term vision, not current priority. Focus is on practical tools that deliver immediate value.

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

### Phase 0: Process 2.2 Mirror (CURRENT)

**Goal:** Create Claude Code version of Opportunity Advancer to validate hybrid approach

**Why this first:**
- Learn patterns before building new agents
- Validate that Claude Code can replicate Notion AI logic
- Establish testing, logging, and execution patterns
- Quick win: 2-3 hours implementation

**Implementation Guide:** `docs/2.2-opportunity-advancer.md` (complete specification)

**Expected Output:**
```bash
# Example usage (future)
claude-code advance-opportunity "Project Name"
# Returns: Markdown report with 5 proposed fields for validation
```

### Phase 1: Weekly Pipeline Digest (NEXT)

**Goal:** First net-new agent (doesn't exist in Notion AI)

**Value:** 30 min/week saved by automating manual pipeline review

**Output:**
- Pipeline Health Score (velocity, stalled count, overdue actions)
- Immediate action items with links and context
- Revenue opportunities (high-momentum deals)
- Weekly agenda by date

**Implementation Guide:** `docs/3.0-weekly-pipeline.md` (complete specification)

**Timeline:** After Phase 0 validated (weeks 2-3)

### Documentation Structure

All technical documentation is in `docs/` folder:

| File | Purpose | When to Use |
|------|---------|-------------|
| `docs/arquitetura.md` | Technical architecture (current + future) | Understand system design |
| `docs/cronograma.md` | 5-phase roadmap (16 weeks) | Planning and tracking |
| `docs/2.2-opportunity-advancer.md` | Phase 0 complete spec | Implementing agent mirror |
| `docs/3.0-weekly-pipeline.md` | Phase 1 complete spec | Implementing digest |
| `plano-atualizado.md` | Strategic overview | High-level decisions |

### Working with Agent Implementations

**When implementing agents:**
1. Always read the spec in `docs/` first (complete processing logic, error handling, test cases)
2. Use MCP tools: `mcp__notion__notion-search`, `mcp__notion__notion-fetch` (pre-approved)
3. Require user approval for write operations: `mcp__notion__notion-update-page`
4. Generate detailed output for user validation (don't update Notion blindly)
5. Log execution steps for debugging

**Oportunidades Database:**
- Collection ID: `collection://201b1882-308d-4524-8a86-6672d5502299`
- Key properties: Task, Biz Funnel, Status, Priority, Next Action, Next Action Date, Update Log, Anotações (relation)

### Future Phases (Not Current Priority)

**Phase 2-4** will add Meeting Prep Briefer, Opportunity Creator, Follow-up Drafter, and eventually workflow orchestration. These are documented in `docs/cronograma.md` but are **not immediate focus**.

**Current philosophy:** Build practical tools that work, validate value, iterate. Autonomous orchestration is long-term vision after foundational agents prove useful.

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

- **Pragmatic approach**: Focus on short-term deliverables and validation, not long-term architecture
- **Data privacy**: Never commit `.env` or share actual API keys
- **Permission model**: Respect the auto-approve list; other operations require user consent
- **Human validation**: Always show proposed changes before updating Notion (dry-run mode by default)
- **Incremental development**: Build, test, iterate. Don't over-engineer before validating value
- **Cross-platform**: Code and configuration must work on macOS, Windows, and Linux

## Development Philosophy

**Priorities (in order):**
1. **Works:** Agent produces useful output
2. **Fast:** Implementation time < 3 hours per agent
3. **Validated:** Tested with real data, user confirms value
4. **Iterated:** Refined based on actual usage patterns

**Anti-patterns to avoid:**
- Building complex orchestration before basic agents work
- Over-engineering for edge cases before validating core use case
- Creating abstractions before patterns emerge (wait for 2-3 agents first)
- Perfectionism over iteration (80% solution that ships > 100% solution that doesn't)
