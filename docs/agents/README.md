# Agent Specifications

Business requirements and specifications for all agents in the system.

## Files

- **[opportunity-advancer-spec.md](opportunity-advancer-spec.md)** - Process 2.2 mirror (operational)
- **[pipeline-digest-spec.md](pipeline-digest-spec.md)** - Weekly digest (Phase 1, planned)

## Purpose

These are **business specifications** (what agents should do, why they exist).

For **technical implementations** (how agents work, SPAR logic), see `/.claude/agents/`

## Distinction: Specs vs Implementations

| Location | Purpose | Contains |
|----------|---------|----------|
| `/docs/agents/` | Business requirements | What, Why, Requirements, Test cases |
| `/.claude/agents/` | Technical implementation | How, SPAR workflow, Code logic |

## When to Read

- Understanding agent requirements
- Comparing agent output with expected behavior
- Designing new agents (use existing specs as template)
- Validating agent implementation matches spec

## Related Documentation

- [/.claude/agents/README.md](../../.claude/agents/README.md) - Agent usage guide
- [QUICK-START.md](../QUICK-START.md) - How to invoke agents
- [architecture/agent-workflow-map.md](../architecture/agent-workflow-map.md) - Agent ecosystem
