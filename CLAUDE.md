# CLAUDE.md

This file provides guidance to Claude Code when working with this CRM project.

## Project Mission

AI-assisted CRM workflow automation using Claude Code + Notion via MCP.

## Current State

**Operational:**
- Process 2.2 (Opportunity Advancer) in Notion AI ✅
- MCP integration with Notion ✅
- Oportunidades database with 11-stage Biz Funnel ✅

**In Progress:**
- Phase 0: Claude Code mirror of Process 2.2 (validate approach)

## Key Constraints

- Respect auto-approve list: `mcp__notion__notion-search`, `mcp__notion__notion-fetch` only
- All writes require user approval
- Update Log must append, never replace
- Biz Funnel stages should not be skipped. Warn when doing so.
- Human validation: always show proposed changes before updating Notion

## Development Approach

**Priorities:**
1. Works: Produces useful output
2. Fast: <3 hours per agent
3. Validated: Tested with real data
4. Iterated: Refined based on usage

**Avoid:**
- Complex orchestration before basics work
- Over-engineering for edge cases
- Abstractions before patterns emerge

## Setup

```bash
# Verify environment
node --version  # Should be >= 18.x
cat .env | grep NOTION_API_KEY  # Should exist

# Test MCP connection
# Open Claude Code - MCP should auto-load
```

## Quick Reference

**Oportunidades Database:** `collection://201b1882-308d-4524-8a86-6672d5502299`

**Biz Funnel Stages:** Marketing → Suspect → Prospect → Contato → Credibilidade → Oferta → Proposta → Negociação → Fechamento → Relacionamento → done!

**Status Options:** In Progress | Waiting Feedback | Scheduled

**Date Property Format:** `date:Next Action Date:start`, `date:Next Action Date:is_datetime: 0`

---

## Essential Reading

**Core Principles:** `docs/PRINCIPLES.md` - 20 principles learned from building this system
**Phase 0 Implementation:** `docs/phase-0-lean-implementation.md`
**Detailed Specs:** See `docs/`
