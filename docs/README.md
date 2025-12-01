# Documentation Index

Welcome to the CRM workflow automation documentation. This guide helps you find what you need quickly.

---

## Quick Links

**🚀 New Here?** Start with [QUICK-START.md](QUICK-START.md) (< 5 minutes)
**❗ Having Issues?** Check [TROUBLESHOOTING.md](TROUBLESHOOTING.md)
**🎯 Understanding Design?** Read [PRINCIPLES.md](PRINCIPLES.md) (27 core principles)

---

## Documentation Map

### Getting Started

| Document | Purpose | Audience | Time |
|----------|---------|----------|------|
| **[QUICK-START.md](QUICK-START.md)** | Primary entry point, common workflows | Users | 5 min |
| **[../README.md](../README.md)** | Environment setup, MCP configuration | Developers | 10 min |
| **[TROUBLESHOOTING.md](TROUBLESHOOTING.md)** | Common issues and solutions | All | As needed |

### Core Philosophy

| Document | Purpose | Audience | Time |
|----------|---------|----------|------|
| **[PRINCIPLES.md](PRINCIPLES.md)** | 27 principles from real implementation | All | 30 min |
| **[architecture/overview.md](architecture/overview.md)** | System architecture, data flow | Developers | 20 min |
| **[architecture/agent-workflow-map.md](architecture/agent-workflow-map.md)** | Agent ecosystem and integration patterns | Developers | 15 min |

### Implementation

| Document | Purpose | Audience | Time |
|----------|---------|----------|------|
| **[implementation/phase-0.md](implementation/phase-0.md)** | Current phase guide | Developers | 15 min |
| **[implementation/roadmap.md](implementation/roadmap.md)** | Phases 0-3 timeline | Product | 10 min |

### Agent Specifications

| Document | Purpose | Audience | Time |
|----------|---------|----------|------|
| **[agents/opportunity-advancer-spec.md](agents/opportunity-advancer-spec.md)** | Business requirements spec | Product | 30 min |
| **[agents/pipeline-digest-spec.md](agents/pipeline-digest-spec.md)** | Future agent spec (Phase 1) | Product | 20 min |
| **[../.claude/agents/README.md](../.claude/agents/README.md)** | Agent usage guide | Users | 10 min |

### Governance

| Document | Purpose | Audience | Time |
|----------|---------|----------|------|
| **[governance/failure-log.md](governance/failure-log.md)** | Known issues, patterns | All | As needed |

### Reference

| Document | Purpose | Audience | Time |
|----------|---------|----------|------|
| **[../CLAUDE.md](../CLAUDE.md)** | Project mission, quick reference | AI Assistants | 3 min |
| **[archive/](archive/)** | Historical docs and session logs | Developers | As needed |

---

## Reading Paths by Role

### 👤 User (Business Operations)

**Goal:** Use agents to automate CRM workflows

**Start here:**
1. [QUICK-START.md](QUICK-START.md) - Learn slash commands and workflows
2. [../.claude/agents/README.md](../.claude/agents/README.md) - Detailed agent usage
3. [TROUBLESHOOTING.md](TROUBLESHOOTING.md) - Bookmark for issues
4. [PRINCIPLES.md](PRINCIPLES.md) - Optional: understand design philosophy

**When to read:**
- Daily: QUICK-START.md for invocations
- As needed: TROUBLESHOOTING.md for errors
- Monthly: governance/failure-log.md for known patterns

---

### 💻 Developer (System Builder)

**Goal:** Understand, maintain, and extend the system

**Start here:**
1. [../README.md](../README.md) - Environment setup
2. [PRINCIPLES.md](PRINCIPLES.md) - **CRITICAL** - Design philosophy
3. [architecture/overview.md](architecture/overview.md) - System architecture
4. [architecture/agent-workflow-map.md](architecture/agent-workflow-map.md) - Agent relationships
5. [implementation/phase-0.md](implementation/phase-0.md) - Current phase

**When modifying system:**
1. Read relevant agent spec in `/docs/agents/`
2. Check agent implementation in `/.claude/agents/`
3. Review PRINCIPLES.md for compliance
4. Update tests in `/tests/`
5. Document in governance/failure-log.md if issues arise

**Deep dives:**
- [archive/2025-11-planning/](archive/2025-11-planning/) - Historical decisions
- [agents/opportunity-advancer-spec.md](agents/opportunity-advancer-spec.md) - Complete spec example

---

### 🤖 AI Assistant (Claude Code)

**Goal:** Follow project conventions and principles

**Essential reading:**
1. [../CLAUDE.md](../CLAUDE.md) - **LOAD THIS FIRST** - Project mission
2. [PRINCIPLES.md](PRINCIPLES.md) - 27 principles to follow
3. [../.claude/skills/crm-data-model/SKILL.md](../.claude/skills/crm-data-model/SKILL.md) - Business rules

**When invoked:**
- Slash command: Load agent spec from `/.claude/agents/`
- Modification request: Check PRINCIPLES.md first
- User feedback: Trigger cs-agent-validator

**Reference:**
- [architecture/overview.md](architecture/overview.md) - System context
- [governance/failure-log.md](governance/failure-log.md) - Avoid known pitfalls

---

## Documentation Status Legend

| Status | Meaning | Example |
|--------|---------|---------|
| ✅ **Current** | Actively maintained, reflects current system | QUICK-START.md, PRINCIPLES.md |
| 🚧 **In Progress** | Partially complete, being developed | Some specs for Phase 1 agents |
| 📚 **Historical** | Archived but valuable for context | archive/sessions/ |
| 🔮 **Planning** | Future reference, not immediately actionable | archive/2025-11-planning/ |

---

## Contribution Guidelines

### Adding New Documentation

**Location rules:**
- **User guides:** `/docs/` root
- **Architecture:** `/docs/architecture/`
- **Specs:** `/docs/agents/`
- **Implementation guides:** `/docs/implementation/`
- **Governance:** `/docs/governance/`
- **Historical:** `/docs/archive/`

**File naming:**
- Use kebab-case: `agent-workflow-map.md`
- Include dates for sessions: `session-2025-11-30-*.md`
- Descriptive names: `mcp-tools-reference.md` not `tools.md`

**Metadata headers:**
```yaml
---
status: Current | Historical | Planning | In Progress
version: 1.0
last_updated: 2025-11-30
language: en
audience: users | developers | ai-assistants
---
```

### Updating Documentation

**When code changes:**
1. Update corresponding spec in `/docs/agents/` if business logic changed
2. Update PRINCIPLES.md if new principle learned
3. Update QUICK-START.md if invocation method changed
4. Update this index if new major document added

**When archiving:**
1. Move to `/docs/archive/` with appropriate subdirectory
2. Update archive/README.md with entry
3. Remove from active links in other docs

---

## Search Tips

**Finding specific topics:**
```bash
# Search all markdown files
grep -r "MCP" --include="*.md" docs/

# Find references to specific agent
grep -r "opportunity-advancer" --include="*.md" docs/

# Search within active docs only (exclude archive)
grep -r "Biz Funnel" --include="*.md" docs/ --exclude-dir=archive
```

**Common search terms:**
- "SPAR" - Agent framework
- "MCP" - Model Context Protocol (Notion integration)
- "Biz Funnel" - Sales pipeline stages
- "Update Log" - Meeting summary field
- "Principle" - Design principles

---

## Document Dependencies

**If you're updating these docs, also check:**

| When updating... | Also check... | Why |
|------------------|---------------|-----|
| QUICK-START.md | TROUBLESHOOTING.md | Keep error guidance consistent |
| PRINCIPLES.md | All agent specs | Agents must comply with principles |
| Agent specs (/docs/agents/) | Agent implementations (/.claude/agents/) | Keep spec and code in sync |
| architecture/overview.md | agent-workflow-map.md | Architecture and flows must align |
| Biz Funnel stages | .claude/skills/crm-data-model/SKILL.md | Business rules live in skill |

---

## Frequently Asked Questions

### "Where do I find X?"

| Looking for... | Check... |
|----------------|----------|
| How to use agents | QUICK-START.md |
| Error solutions | TROUBLESHOOTING.md |
| Why the system works this way | PRINCIPLES.md |
| Technical architecture | architecture/overview.md |
| Business requirements | agents/*-spec.md |
| Agent implementation code | ../.claude/agents/*.md |
| Known bugs | governance/failure-log.md |
| Historical context | archive/ |
| Environment setup | ../README.md |

### "Which file should I edit?"

| To change... | Edit... |
|--------------|---------|
| Agent behavior | .claude/agents/[agent-name].md |
| Business rules | .claude/skills/crm-data-model/SKILL.md |
| Permissions | .claude/settings.local.json |
| User guide | QUICK-START.md or TROUBLESHOOTING.md |
| Business requirements | docs/agents/[agent-name]-spec.md |

### "Documentation seems outdated"

1. Check `last_updated` in file header
2. Compare with git log: `git log --oneline docs/[filename]`
3. If > 1 month old and system changed, consider updating
4. Add to `last_updated` when you make changes

---

## Next Steps

Choose your path:

**🚀 Ready to use the system?**
→ Go to [QUICK-START.md](QUICK-START.md)

**🐛 Encountering issues?**
→ Check [TROUBLESHOOTING.md](TROUBLESHOOTING.md)

**🏗️ Building or modifying?**
→ Read [PRINCIPLES.md](PRINCIPLES.md) first, then [architecture/overview.md](architecture/overview.md)

**📖 Understanding the approach?**
→ Start with [PRINCIPLES.md](PRINCIPLES.md), then [architecture/agent-workflow-map.md](architecture/agent-workflow-map.md)

---

**Last Updated:** 2025-11-30 (Phase 1 documentation housekeeping)
