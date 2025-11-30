# Documentation Archive

This directory contains deprecated, historical, and reference documentation that is no longer actively used but preserved for context and learning.

## Structure

```
archive/
├── 2025-11-planning/  # Strategic planning and architectural decisions
└── sessions/          # Session logs and investigations
```

---

## Archived Content

### 2025-11-planning/ - Strategic Planning Documents

**Why archived:** These documents served their purpose during initial planning. Key insights have been extracted into PRINCIPLES.md and current documentation.

| File | Original Purpose | Why Archived | Key Learnings Extracted To |
|------|------------------|--------------|----------------------------|
| `plano-atualizado.md` | Portuguese strategic roadmap | Superseded by English docs in /implementation | PRINCIPLES.md, roadmap.md |
| `architectural-plans-skills-vs-subagents.md` | Comparison of Plan A vs Plan B architecture | Decision made: Plan A (Skills-First) adopted | PRINCIPLES.md (Principle 2) |
| `spar-architecture-future-reference.md` | Deep dive into SPAR framework | Reference material, not immediately actionable for Phase 0 | PRINCIPLES.md, agentic-workflow-map.md |
| `notion-setup-ai-recommendation.md` | Notion AI configuration recommendations | Notion-side configuration, outside Claude Code scope | N/A (Notion-specific) |

### sessions/ - Session Logs and Investigations

**Why archived:** Session logs captured valuable learnings that have been consolidated into PRINCIPLES.md and governance/failure-log.md.

| File | Date | Purpose | Key Learnings Extracted To |
|------|------|---------|----------------------------|
| `session-2025-11-21-crm-enhancement.md` | 2025-11-21 | CRM enhancement session | PRINCIPLES.md |
| `session-2025-11-21-153000-summary.md` | 2025-11-21 | Session summary | PRINCIPLES.md |
| `session-2025-11-22-agent-review.md` | 2025-11-22 | Agent review session | PRINCIPLES.md, failure-log.md |
| `session-2025-11-25-post-search-filtering.md` | 2025-11-25 | Post-search filtering implementation | opportunity-advancer.md |
| `logs/2025-11-23-session.md` | 2025-11-23 | Session log | PRINCIPLES.md |
| `search-memory-analysis.md` | 2025-11-23+ | Search memory pattern analysis | opportunity-advancer.md (search protocol) |

---

## How to Use This Archive

### When to Reference
- Understanding historical context for current decisions
- Learning from past architectural discussions
- Reviewing evolution of principles and patterns
- Troubleshooting similar issues that occurred before

### When NOT to Reference
- For current implementation guidance (use /docs instead)
- For active agent specifications (use /.claude/agents instead)
- For operational procedures (use QUICK-START.md, PRINCIPLES.md)

---

## Archival Policy

**Documents are archived when:**
1. Their purpose has been fulfilled
2. Key learnings have been extracted to active documentation
3. They would cause confusion if left in main docs
4. They represent historical decisions that have been made

**Documents are NOT deleted because:**
- Git history is valuable but hard to search
- Context helps understand "why" decisions were made
- Future reference may reveal patterns
- Institutional knowledge preservation

---

## Last Updated
2025-11-30 - Initial archive creation during Phase 1 documentation housekeeping
