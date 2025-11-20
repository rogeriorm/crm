# Phase 0: Lean Implementation - Opportunity Advancer

**Objective:** Create Claude Code mirror of Process 2.2 to validate hybrid approach
**Time:** 2-3 hours
**Status:** Ready to implement

---

## Why This Matters

Process 2.2 already works in Notion AI. Creating a Claude Code version gives you:
- **Versionamento** (track changes in git)
- **Testing** (validate before production)
- **Logging** (understand decisions)
- **Foundation** (patterns for future agents)

---

## Quick Setup

### 1. Create Sub-Agent File (5 min)

Create `.claude/agents/opportunity-advancer.md`:

```yaml
name: opportunity-advancer
description: "Analyze opportunities after meetings and recommend 5 field updates. Use PROACTIVELY when user mentions analyzing or updating opportunity progress."
model: sonnet
tools:
  - mcp__notion__notion-search
  - mcp__notion__notion-fetch
  - mcp__notion__notion-update-page
permissionMode: ask

system_instructions: |
  ## Mission
  After client meetings, update 5 opportunity fields to keep pipeline moving.

  ## Process

  **SENSE (Load Context):**
  1. Search Oportunidades by name
  2. Fetch current properties: Biz Funnel, Status, Priority, Update Log
  3. Load all Anotações (meeting notes)
  4. Find most recent by Date property
  5. Extract content from <transcript>, <summary>, <notes> tags

  **PLAN (Analyze & Generate):**
  1. Analyze meeting for key decisions, commitments, signals
  2. Generate Update Log: 1 sentence, max 10 words, format "DD/MM: [action/result]"
  3. Determine Next Action: verb + object (specific and actionable)
  4. Calculate Next Action Date: last interaction + 7 days (or client deadline if mentioned)
  5. Detect Biz Funnel advancement:
     - Credibilidade → Oferta: client asks pricing
     - Oferta → Proposta: client requests formal proposal
     - Proposta → Negociação: client negotiating terms
     - Never skip stages
  6. Set Status:
     - "In Progress" = user has action
     - "Waiting Feedback" = awaiting client
     - "Scheduled" = meeting booked

  **ACT (Present & Update):**
  1. Show proposed 5 fields in markdown
  2. Wait for user approval
  3. Update Notion (append to Update Log, don't replace)

  **REFLECT (Validate):**
  1. Did update succeed?
  2. Do values make sense?

  ## Constraints
  - Update Log ≤10 words
  - Never skip Biz Funnel stages
  - Always append Update Log
  - Only "Waiting Feedback" if truly waiting on client

  ## Output Format
  ```markdown
  # Opportunity Update: [Name]

  **Last Interaction:** [Date]
  **Current Stage:** [Stage] → [New Stage if changed]

  ## Proposed Updates
  1. **Update Log:** [DD/MM: sentence]
  2. **Next Action:** [verb + object]
  3. **Next Action Date:** [YYYY-MM-DD]
  4. **Biz Funnel:** [stage] (Changed: yes/no)
  5. **Status:** [In Progress | Waiting Feedback | Scheduled]

  **Apply updates?** [Y/N]
  ```
```

### 2. Create CRM Data Model Skill (10 min)

Create `.claude/skills/crm-data-model/SKILL.md`:

```markdown
# CRM Data Model

**Activate when:** User mentions opportunities, pipeline, Biz Funnel

## Oportunidades Database

**ID:** `collection://201b1882-308d-4524-8a86-6672d5502299`

## Properties

- **Task** (title): Opportunity name
- **Biz Funnel** (select): Marketing → Suspect → Prospect → Contato → Credibilidade → Oferta → Proposta → Negociação → Fechamento → Relacionamento → done!
- **Status** (select): In Progress | Waiting Feedback | Scheduled | On Hold | Lost | Won
- **Priority** (select): High | Medium | Low
- **Update Log** (text): Chronological updates (newest first)
- **Next Action** (text): Specific next step
- **Next Action Date** (date): When to act
- **Anotações** (relation): Meeting notes with transcripts
- **Offline Notes** (text): Manual notes
- **Days in Stage** (formula): Time in current stage
- **Days Until Due** (formula): Time to deadline

## Advancement Signals

- **Credibilidade → Oferta:** Client wants pricing/proposal
- **Oferta → Proposta:** Client requests formal proposal
- **Proposta → Negociação:** Client negotiating terms/price
- **Negociação → Fechamento:** Client confirmed purchase/signed

## Status Rules

- **In Progress:** User must take action
- **Waiting Feedback:** Awaiting client response
- **Scheduled:** Next meeting has confirmed date/time
```

### 3. Optimize CLAUDE.md (15 min)

Replace current CLAUDE.md with:

```markdown
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

- Respect auto-approve list: `notion-search`, `notion-fetch` only
- All writes require user approval
- Update Log must append, never replace
- Biz Funnel stages cannot be skipped

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

---

For detailed specs: See `docs/`
```

---

## Test the Agent (1-2 hours)

### Test Case 1: Standard Update

```
User: "Analyze opportunity progress for [real opportunity name]"

Expected:
1. Agent searches Oportunidades
2. Loads latest Anotação
3. Generates 5 fields
4. Presents for approval
5. Updates Notion after confirmation
```

### Test Case 2: Stage Advancement

Pick an opportunity that clearly advanced stages.

```
User: "Update [opportunity where client requested proposal]"

Expected:
- Biz Funnel: Credibilidade → Oferta (Changed: yes)
- Reasoning: "Client requested pricing information"
```

### Test Case 3: Waiting on Client

Pick an opportunity awaiting client response.

```
User: "Update [opportunity waiting on client]"

Expected:
- Status: Waiting Feedback
- Next Action: Something client must do (not user)
```

### Compare with Notion AI

For 3-5 real opportunities:
1. Run Claude Code agent
2. Run Notion AI Process 2.2
3. Compare outputs
4. Document differences
5. Refine if <80% match

---

## Acceptance Criteria

- [ ] Agent processes opportunity in <30 seconds
- [ ] 5 fields generated match Notion AI ≥80% of time
- [ ] Update Log is ≤10 words
- [ ] Next Action is specific and actionable
- [ ] Biz Funnel advancement logic makes sense
- [ ] User can easily approve/reject updates
- [ ] Notion update succeeds without errors

---

## Troubleshooting

**Agent not finding opportunity:**
- Check exact spelling (case-sensitive)
- Try URL instead of name
- Verify opportunity exists in database

**No meeting data found:**
- Ensure Anotações relation populated
- Check Offline Notes as fallback
- Verify <transcript> tags in notes

**Wrong stage advancement:**
- Review advancement signals in SKILL.md
- Adjust confidence threshold
- User can reject and agent learns (future)

**Update failed:**
- Check NOTION_API_KEY in .env
- Verify MCP server running
- Ensure user approved write operation

---

## Next Steps After Phase 0

Once this works:

**Phase 1 (Weeks 2-3):** Weekly Pipeline Digest
- Query all active opportunities
- Calculate health metrics
- Generate actionable digest
- Saves ~30 min/week

**Decision Gate:** If Phase 0 validates approach, proceed to Phase 1

---

## Time Estimate Breakdown

- Create agent file: 5 min
- Create skill file: 10 min
- Optimize CLAUDE.md: 15 min
- Test with 3 opportunities: 30-45 min
- Compare with Notion AI: 30-45 min
- Refine based on results: 15-30 min
- Document findings: 15 min

**Total: 2-3 hours**

---

**Ready to start?** Create the files above and test with a real opportunity.
