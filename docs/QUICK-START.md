# Quick Start Guide

**Get started with CRM workflow automation in under 5 minutes.**

---

## What This System Does

AI-assisted CRM workflow automation using Claude Code + Notion via MCP:
- **Analyze opportunities** after client meetings
- **Recommend field updates** to keep pipeline moving
- **Track patterns** across interactions
- **Maintain governance** with human validation gates

**Visual Overview:** See [architecture/diagrams/system-overview.mmd](architecture/diagrams/system-overview.mmd) for system architecture.

---

## Quick Invocation Reference

### Slash Commands (Recommended - Deterministic)

```
/analyze-opportunity <opportunity-name>
```

**Example:**
```
/analyze-opportunity Musashi - 600k
```

**Why use slash commands?**
- Reliable, deterministic triggers
- MCP tools guaranteed available
- Discoverable via `/help`
- See Principle 27 in PRINCIPLES.md

### Natural Language (Best-Effort Fallback)

```
"Analyze opportunity [Name]"
"Update progress for [Opportunity Name]"
```

**Note:** Natural language invocation works but is less reliable than slash commands due to LLM interpretation variability.

---

## Common Workflows

### 1. Analyze Opportunity After Meeting

**When:** You just had a client meeting and added notes to Notion

**Steps:**
1. Ensure meeting notes are in Anotações (related pages) or Offline Notes field
2. In Claude Code: `/analyze-opportunity <exact opportunity name>`
3. Agent searches database and loads 2-3 recent interactions
4. Agent analyzes and proposes 6 field updates
5. Review proposed changes (shown with Notion URL for validation)
6. Approve or reject (`Y/N` or `E` to edit)
7. Agent updates Notion only after approval

**Time:** ~30-60 seconds

**Visual Workflow:** See [architecture/diagrams/agent-workflow.mmd](architecture/diagrams/agent-workflow.mmd) for SPAR sequence diagram.

**Example:**
```
You: /analyze-opportunity Ferreira Costa - Sávio e Juarez Guimarães

Agent: [Searches and validates result with you]
Agent: Is this correct? Y/N
You: Y

Agent: [Loads meeting notes]
Agent: [Analyzes patterns]

# Opportunity Update: Ferreira Costa

Proposed Updates:
1. Update Log: 21/11: cliente confirmou interesse técnico
2. Next Action: Aguardar confirmação de budget Q1
3. Next Action Date: 2025-11-28
4. Biz Funnel: Credibilidade (no change)
5. Status: Waiting Feedback
6. AI Recommendation: Cliente mencionou budget em 2 interações...

Apply? Y/N/E

You: Y

Agent: ✅ Updated successfully!
```

---

### 2. Compare Claude Code vs Notion AI

**When:** Validating Phase 0 hybrid approach

**Steps:**
1. Pick 3-5 real opportunities with recent meetings
2. Run Claude Code: `/analyze-opportunity [Name]`
3. Note the 6 proposed fields
4. Run Notion AI: "Atualize o avanço da oportunidade [Nome]"
5. Compare outputs field-by-field
6. Document differences and patterns

**Target:** >80% match between outputs

---

### 3. Implement System Change

**When:** You have a GitHub issue or improvement to implement

**Steps:**
1. Ensure GitHub issue exists (or describe the change)
2. In Claude Code: `"Implement issue #[number]"` or `"Implement [description]"`
3. system-implementer agent reads issue and scans current state
4. Agent presents implementation plan with file changes
5. Review and approve plan
6. Agent implements changes sequentially with approval gates
7. Agent validates completion and provides commit message
8. Review changes and commit

**Example:**
```
You: Implement issue #19

Agent: [Reads issue, scans PRINCIPLES.md, maps dependencies]

## Implementation Plan: Archive deprecated documentation

Files to modify:
1. Create docs/archive/ structure
2. Move 9 deprecated files
3. Create archive/README.md
...

Estimated: 1-2 hours
Proceed? Y/N

You: Y

Agent: [Implements step-by-step with checkpoints]
Agent: ✅ Complete. Ready to commit.
```

---

## Available Agents

### Business Operations

| Agent | Slash Command | Natural Language | Purpose |
|-------|---------------|------------------|---------|
| **opportunity-advancer** | `/analyze-opportunity <name>` | "Analyze opportunity [Name]" | Analyzes meetings, recommends 6 field updates |

### Technical Enablement

| Agent | Invocation | Purpose |
|-------|------------|---------|
| **system-implementer** | "Implement issue #N" | Executes validated system modifications |
| **cs-agent-validator** | Auto-triggered on feedback | Validates feedback vs specs, creates issues |
| **ai-enablement-reviewer** | "Review agent configs" | Reviews for token efficiency, business value |
| **business-architect** | "Design process for..." | Designs business processes using SPAR |

---

## Decision Matrix: Which Agent to Use?

| Scenario | Agent | Invocation |
|----------|-------|------------|
| Just had client meeting | opportunity-advancer | `/analyze-opportunity <name>` |
| Want to compare with Notion AI | opportunity-advancer | `/analyze-opportunity <name>` |
| Have GitHub issue to implement | system-implementer | "Implement issue #N" |
| Agent isn't working right | cs-agent-validator | Describe what went wrong (auto-triggers) |
| Need to optimize agent prompts | ai-enablement-reviewer | "Review opportunity-advancer config" |
| Designing new workflow | business-architect | "Design process for weekly pipeline digest" |

---

## Troubleshooting Quick Tips

### "Opportunity not found"
- ✅ Use exact name from Notion (case-sensitive)
- ✅ Try full opportunity name
- ✅ Or use Notion URL: `/analyze-opportunity https://notion.so/...`

### "No meeting data to process"
- ✅ Add content to Anotações (related pages)
- ✅ Or populate Offline Notes field
- ✅ Verify Anotação has content in transcript/summary/notes

### "MCP tool not available"
- ✅ Use slash command instead of natural language
- ✅ Check `.env` has valid NOTION_API_KEY
- ✅ Verify MCP server auto-loaded (shows in Claude Code startup)

### "Agent suggests wrong Biz Funnel stage"
- ✅ You can reject the update
- ✅ Agent won't skip stages without warning
- ✅ Check advancement signals in `.claude/skills/crm-data-model/SKILL.md`

### Need more help?
- See [TROUBLESHOOTING.md](TROUBLESHOOTING.md) for detailed guides
- Check [governance/failure-log.md](governance/failure-log.md) for known issues
- Review [PRINCIPLES.md](PRINCIPLES.md) for system design philosophy

---

## Configuration & Customization

### Adjust Agent Behavior
**File:** `.claude/agents/opportunity-advancer.md`

Edit to change:
- SENSE/PLAN/ACT/REFLECT logic
- Output format
- Constraints (e.g., Update Log max length)

### Adjust Business Rules
**File:** `.claude/skills/crm-data-model/SKILL.md`

Edit to change:
- Biz Funnel stages (11 stages currently)
- Advancement signals (when to move stages)
- Status decision rules
- Field definitions

### Adjust Permissions
**File:** `.claude/settings.local.json`

Controls:
- Auto-approved operations (read-only by default)
- MCP server configuration
- Tool permissions

---

## What's Next?

### Phase 0 (Current) - Validate Approach
- ✅ opportunity-advancer agent operational
- ✅ Compare with Notion AI Process 2.2
- 🚧 Achieve >80% match on outputs

### Phase 1 (Weeks 2-3) - More Business Agents
- Weekly Pipeline Digest
- Meeting Prep Briefer
- Follow-up Drafter

### Phase 2 (5+ agents) - Governance Infrastructure
- Agent coordination
- Shared governance conventions
- Automated testing

### Phase 3 (Complex orchestration)
- Multi-agent workflows
- Event-driven triggers
- Dedicated governance service

---

## Links to Deeper Documentation

**Architecture & Design:**
- [PRINCIPLES.md](PRINCIPLES.md) - 27 principles learned from building this system
- [architecture/overview.md](architecture/overview.md) - System architecture and data flow
- [architecture/agent-workflow-map.md](architecture/agent-workflow-map.md) - Agent ecosystem map

**Implementation:**
- [implementation/phase-0.md](implementation/phase-0.md) - Current phase guide
- [implementation/roadmap.md](implementation/roadmap.md) - Phases 0-3 timeline

**Agents:**
- [agents/opportunity-advancer-spec.md](agents/opportunity-advancer-spec.md) - Complete spec
- [.claude/agents/README.md](../.claude/agents/README.md) - Agent usage guide

**Governance:**
- [governance/failure-log.md](governance/failure-log.md) - Known issues and patterns

**Setup:**
- [../README.md](../README.md) - Environment setup and MCP configuration
- [../CLAUDE.md](../CLAUDE.md) - Project mission and quick reference

---

## Getting Help

**For errors:** Check [TROUBLESHOOTING.md](TROUBLESHOOTING.md)
**For concepts:** Read [PRINCIPLES.md](PRINCIPLES.md)
**For architecture:** See [architecture/overview.md](architecture/overview.md)
**For feedback:** Provide feedback naturally - cs-agent-validator auto-triggers

**Time to first success:** < 5 minutes with this guide 🚀
