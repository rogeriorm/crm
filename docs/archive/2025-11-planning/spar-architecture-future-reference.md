# SPAR-Based Agent Architecture - Future Reference

**Version:** 1.0
**Date:** 2025-11-20
**Purpose:** Comprehensive architectural framework for scaling agent-based CRM workflows

---

## Executive Summary

This document integrates learnings from:
- Claude Code course (Skills, Sub-agents, MCP patterns)
- SPAR framework (Sense-Plan-Act-Reflect cycle)
- Autonomy progression levels (0-5 scale)
- Andrew Ng's agentic AI patterns
- Token optimization strategies

**Current State:** Level 3 autonomy (LLM Workflows via agents)
**Target State:** Level 4 autonomy (Semi-autonomous agents with learning)

**Token Efficiency:** Sub-agent architecture provides 84% token reduction (400K vs 2.5M over 100 operations)

---

## SPAR Framework Foundation

### What is SPAR?

**S**ense → **P**lan → **A**ct → **R**eflect

A universal cycle for agent design that works regardless of technology stack:

```
┌─────────────────────────────────────────────────────────────┐
│                    SPAR CYCLE                               │
│                                                             │
│  [S] SENSE                                                  │
│  - Perceive environment                                     │
│  - Collect relevant context                                 │
│  - Load necessary data                                      │
│                                                             │
│  [P] PLAN                                                   │
│  - Reason about the problem                                 │
│  - Determine action sequence                                │
│  - Consider constraints                                     │
│                                                             │
│  [A] ACT                                                    │
│  - Execute planned actions                                  │
│  - Use tools and APIs                                       │
│  - Produce outputs                                          │
│                                                             │
│  [R] REFLECT                                                │
│  - Evaluate results                                         │
│  - Decide if replanning needed                              │
│  - Learn from feedback (Level 4+)                           │
│                                                             │
│  ↓ (Loop back or exit)                                      │
└─────────────────────────────────────────────────────────────┘
```

**Key Insight:** Reflection with external feedback > reflection in vacuum

---

## Autonomy Progression Levels

### Level 0-5 Framework

| Level | Name | Characteristics | CRM Example |
|-------|------|-----------------|-------------|
| **0** | Manual | Humans do everything | Manually updating Notion fields after meetings |
| **1** | Rule-based | Deterministic automation | "If status=X, then send email Y" |
| **2** | IPA | ML/NLP pattern recognition | Extract entities from emails |
| **3** | LLM Workflows | Agents execute defined workflows | **← CURRENT: Process 2.2 Opportunity Advancer** |
| **4** | Semi-autonomous | Learning, cross-session memory | **← TARGET: Agents learn from corrections** |
| **5** | Fully autonomous | AGI-level reasoning | Self-improving agents (future) |

### Current State: Level 3

**What we have:**
- Process 2.2 uses LLM reasoning to analyze meeting transcripts
- Generates 5 fields (Update Log, Next Action, Biz Funnel, Status, Date)
- User confirms before updates applied

**What we're missing for Level 4:**
- Cross-session memory (agent doesn't remember corrections)
- Confidence scoring (no "I'm 80% sure this should be Proposta stage")
- Autonomous error recovery (agent can't fix mistakes without user)
- Learning from feedback (corrections don't improve future behavior)

---

## Andrew Ng's Agentic Patterns

### 1. Reflection Pattern
**Definition:** Agent critiques its own work before presenting to user

**CRM Application:**
```
Agent generates 5 fields → Self-critique:
- "Is Update Log <10 words?" ✓
- "Is Next Action specific enough?" ✓
- "Does Biz Funnel advancement make sense?" ✓
→ Present to user
```

### 2. Tool Use Pattern
**Definition:** LLM decides which functions/APIs to call based on task

**CRM Application:**
```
User: "Prepare for meeting with Projeto SiDi"
Agent decides:
1. mcp__notion__notion-search ("Projeto SiDi")
2. mcp__notion__notion-fetch (load opportunity)
3. mcp__notion__notion-fetch (load last 3 Anotações)
4. Generate briefing with SPIN questions
```

### 3. Planning Pattern
**Definition:** LLM breaks down complex task into steps

**CRM Application:**
```
Task: "Advance all stalled opportunities"
Agent plan:
1. Query: Days in Stage > 14
2. For each opportunity:
   - Load last Anotação
   - If no Anotação: suggest Next Action
   - If has Anotação: run Opportunity Advancer
3. Generate summary report
```

### 4. Multi-agent Pattern
**Definition:** Specialized agents collaborate

**CRM Application:**
```
Orchestrator detects: New Anotação
→ Calls Opportunity Advancer (advance pipeline)
→ Detects: Stage changed Credibilidade → Proposta
→ Calls Meeting Prep Briefer (prepare next meeting)
→ Calls Follow-up Drafter (thank-you email)
```

---

## Token Optimization Strategy

### Problem: Context Accumulation

**Main Thread (Skills-First):**
```
Operation 1: 4K context
Operation 2: 4K + 20K (loaded skills) = 24K
Operation 3: 24K + 15K (analysis) = 39K
Operation 4: 39K + 12K = 51K
...
Operation 100: ~2.5M tokens accumulated
```

**Sub-Agent Architecture:**
```
Call Agent 1: 4K context (fresh)
Call Agent 2: 4K context (fresh)
Call Agent 3: 4K context (fresh)
...
Call Agent 100: 4K × 100 = 400K tokens total
```

**Reduction: 84% fewer tokens** (400K vs 2.5M)

### CLAUDE.md Optimization

**Before (223 lines):**
- Full architectural vision
- Orchestration details
- All agent descriptions
- Complete data model

**After (50 lines):**
- Project mission only
- Current implementation status
- Essential constraints
- Progressive disclosure via skills

**Reduction: 85% fewer tokens**

### Skills with Progressive Disclosure

**Structure:**
```
.claude/skills/crm-data-model/
├── SKILL.md (concise, always loaded)      ← 500 tokens
└── reference.md (detailed, on-demand)     ← 5K tokens
```

**Loading:**
- Discovery: SKILL.md loaded automatically (500 tokens)
- Deep dive: reference.md loaded only when needed (5K tokens)
- **90% of content loaded on-demand**

---

## Sub-Agent Architecture Design

### SPAR-Structured Sub-Agent Template

```yaml
# agent-name.md

name: agent-name
description: "One-line purpose. Use PROACTIVELY when [trigger condition]."
model: sonnet  # or haiku for simple tasks
tools:
  - mcp__notion__notion-search
  - mcp__notion__notion-fetch
  # Add write tools only if needed
permissionMode: ask  # or approve for read-only

system_instructions: |
  ## Mission
  [Clear, focused objective]

  ## SPAR Cycle

  ### [S] SENSE
  - Load context: [what data to fetch]
  - Use skills: [which skills to activate]
  - Gather: [what information]

  ### [P] PLAN
  - Analyze: [what to look for]
  - Decide: [what decisions to make]
  - Generate: [what outputs]

  ### [A] ACT
  - Execute: [what actions]
  - Update: [what to modify]
  - Output: [what format]

  ### [R] REFLECT
  - Validate: [what to check]
  - Self-critique: [what to verify]
  - Fallback: [what if errors]

  ## Constraints
  - [Important limitations]
  - [Edge cases to handle]

  ## Output Format
  ```
  [Expected output structure]
  ```
```

### Real Example: Opportunity Advancer

```yaml
name: opportunity-advancer
description: "Advance opportunities through funnel after meetings. Use PROACTIVELY when user mentions analyzing or updating opportunity progress."
model: sonnet
tools:
  - mcp__notion__notion-search
  - mcp__notion__notion-fetch
  - mcp__notion__notion-update-page
permissionMode: ask

system_instructions: |
  ## Mission
  After client meetings, analyze transcripts and update 5 opportunity fields to keep pipeline moving.

  ## SPAR Cycle

  ### [S] SENSE
  - Search Oportunidades database by name
  - Fetch opportunity properties: Biz Funnel, Status, Priority, Days in Stage
  - Load all Anotações (meeting notes), identify most recent
  - Extract content from <transcript>, <summary>, <notes> tags
  - Use skills: crm-data-model, funnel-analysis

  ### [P] PLAN
  - Analyze meeting content for key decisions, commitments, signals
  - Detect Biz Funnel advancement (keywords: "proposta", "pricing", "contract")
  - Generate concise Update Log (1 sentence, max 10 words)
  - Determine specific Next Action (verb + object)
  - Calculate Next Action Date (last interaction + 7 days, or client deadline)
  - Set Status based on who has the ball (user vs client)

  ### [A] ACT
  - Present 5 proposed fields for user review
  - After approval: update Notion page
  - Append Update Log (don't replace existing)

  ### [R] REFLECT
  - Did update succeed? (check API response)
  - Do values make sense? (sanity check)
  - Should trigger follow-up? (if stage advanced)

  ## Constraints
  - Update Log must be ≤10 words
  - Never skip Biz Funnel stages (e.g., can't go Credibilidade → Negociação)
  - Status "Waiting Feedback" only if literally waiting on client
  - Always append to Update Log, never replace

  ## Output Format
  ```markdown
  # Opportunity Update: [Name]

  **Last Interaction:** [Date]
  **Current Stage:** [Stage] → [New Stage if changed]

  ## Proposed Updates
  1. Update Log: [one sentence]
  2. Next Action: [verb + object]
  3. Next Action Date: [YYYY-MM-DD]
  4. Biz Funnel: [stage] (Changed: yes/no)
  5. Status: [In Progress | Waiting Feedback | Scheduled]

  [Apply updates? Y/N]
  ```
```

---

## Consulting Service Framework

### Technology-Agnostic Design Process

**Phase 1: Business Understanding (Universal)**
1. Map current processes and pain points
2. Assess autonomy level (0-5 scale)
3. Identify agent opportunities using SPAR lens
4. Design agent missions, objectives, constraints
5. Create technology-agnostic specifications

**Phase 2: Implementation (Tech Stack Dependent)**

| Client Scale | Tech Stack | Cost Range | Implementation Time |
|--------------|-----------|------------|---------------------|
| **Small (<10 users)** | Claude Code + Notion | $5K-15K | 2-4 weeks |
| **Medium (10-100 users)** | Claude Code + Custom DB | $15K-50K | 4-8 weeks |
| **Enterprise (100+ users)** | Custom Agent Framework | $50K-200K | 8-16 weeks |

**Key Insight:** Business design remains the same; only implementation technology changes.

### Pricing Model

**Discovery & Design (Fixed Price):**
- Process mapping workshop: $2K-5K
- Autonomy assessment: $1K-2K
- Agent design specifications: $3K-8K
- **Total Discovery:** $6K-15K

**Implementation (Time & Materials):**
- Small: $5K-15K (Claude Code)
- Medium: $15K-50K (Hybrid)
- Enterprise: $50K-200K (Custom)

**Ongoing (Retainer):**
- Agent maintenance: $1K-3K/month
- New agent development: $2K-5K/agent
- Optimization & training: $500-2K/month

---

## Evolution Roadmap: Level 3 → Level 4

### Current State (Level 3)

**What works:**
- Process 2.2 analyzes transcripts and generates 5 fields
- User confirms updates before applying
- Pipeline stays current after meetings

**Limitations:**
- Agent doesn't remember corrections ("always set Status=Waiting when client must respond")
- No confidence scoring ("I'm 90% sure this is Proposta stage")
- Can't recover from errors autonomously

### Target State (Level 4)

**Enhancements:**

1. **Cross-Session Memory**
   ```yaml
   # Stored in .claude/memory/opportunity-advancer.json
   {
     "patterns_learned": [
       {
         "rule": "When client says 'send me pricing', advance to Oferta",
         "confidence": 0.95,
         "learned_from": 12  # 12 corrections reinforced this
       }
     ],
     "user_preferences": {
       "status_waiting_feedback": "Only when explicitly waiting on client response",
       "next_action_date_buffer": 7  # days
     }
   }
   ```

2. **Confidence Scoring**
   ```markdown
   ## Proposed Updates
   1. Update Log: "18/11: cliente aprovou proposta técnica" [Confidence: 95%]
   2. Next Action: "Enviar proposta comercial" [Confidence: 90%]
   3. Biz Funnel: Credibilidade → Proposta [Confidence: 85% - Review recommended]
   4. Status: In Progress [Confidence: 100%]
   ```

3. **Autonomous Error Recovery**
   ```
   Agent detects: Last 3 updates had Biz Funnel corrections
   → Reflection: "My stage detection may be too aggressive"
   → Adjustment: Increase confidence threshold for stage changes
   → Future: Only suggest stage change if confidence >90% (was 75%)
   ```

4. **Learning from Feedback**
   ```
   User corrects: "Status should be Waiting Feedback, not In Progress"
   Agent learns: Extract pattern from correction
   Pattern: "Next Action = 'Aguardar resposta do cliente'" → Status = Waiting Feedback
   Apply: Update memory, use in future decisions
   ```

---

## Implementation Patterns

### Pattern 1: Advancement Agent (Process 2.2)

**SPAR Mapping:**
- **Sense:** Load opportunity + latest meeting notes
- **Plan:** Analyze transcript, detect signals, generate 5 fields
- **Act:** Update Notion properties (after approval)
- **Reflect:** Validate update succeeded, check if stage change triggers next agent

**Token Impact:** 4K per invocation (fresh context each time)

### Pattern 2: Digest Agent (Weekly Pipeline)

**SPAR Mapping:**
- **Sense:** Query all active opportunities, load metrics
- **Plan:** Calculate health score, identify risks/opportunities
- **Act:** Generate structured digest report
- **Reflect:** User feedback on action items → refine thresholds

**Token Impact:** 8K per invocation (more data to process)

### Pattern 3: Briefing Agent (Meeting Prep)

**SPAR Mapping:**
- **Sense:** Load opportunity history, last 3 Anotações
- **Plan:** Generate SPIN questions adapted to Biz Funnel stage
- **Act:** Output briefing document
- **Reflect:** Was briefing useful? Track usage rate

**Token Impact:** 5K per invocation

### Pattern 4: Creator Agent (Opportunity Entry)

**SPAR Mapping:**
- **Sense:** Gather input (lead source, client, context)
- **Plan:** Fill template, link parent task, copy contacts
- **Act:** Create Notion page with initial properties
- **Reflect:** Validate required fields populated

**Token Impact:** 3K per invocation (simple data entry)

### Pattern 5: Orchestrator Agent (Multi-Agent)

**SPAR Mapping:**
- **Sense:** Detect event (new Anotação created)
- **Plan:** Determine workflow (Advance → Prep → Follow-up?)
- **Act:** Call sub-agents in sequence
- **Reflect:** Did workflow complete? Any errors?

**Token Impact:** 2K (orchestration) + sum of sub-agent calls

---

## Skills Library Design

### Skill Structure

```
.claude/skills/
├── crm-data-model/
│   ├── SKILL.md (500 tokens, always loaded)
│   └── reference.md (5K tokens, on-demand)
├── funnel-analysis/
│   ├── SKILL.md (400 tokens)
│   └── stage-signals.md (2K tokens)
├── email-templates/
│   ├── SKILL.md (300 tokens)
│   └── templates/ (10K tokens, on-demand)
└── spin-questions/
    ├── SKILL.md (400 tokens)
    └── questions-by-stage.md (3K tokens)
```

### Example: crm-data-model/SKILL.md

```markdown
# CRM Data Model

**Activate when:** User mentions opportunities, pipeline, Biz Funnel, or Notion CRM

## Core Database

**Oportunidades** (`collection://201b1882-308d-4524-8a86-6672d5502299`)

## Key Properties

- **Task** (title): Opportunity name
- **Biz Funnel** (select): Sales stage (11 stages)
- **Status** (select): Workflow state
- **Priority** (select): High/Medium/Low
- **Update Log** (text): Chronological updates
- **Next Action** (text): Specific next step
- **Next Action Date** (date): When to act
- **Anotações** (relation): Meeting notes
- **Days in Stage** (formula): Time in current stage
- **Days Until Due** (formula): Time to deadline

## Biz Funnel Stages

Marketing → Suspect → Prospect → Contato → **Credibilidade** → Oferta → **Proposta** → Negociação → Fechamento → Relacionamento → done!

Bold stages = most common for active deals

## Status Options

- **In Progress**: User has action to take
- **Waiting Feedback**: Awaiting client response
- **Scheduled**: Next meeting booked
- **On Hold**: Temporarily paused
- **Lost**: Deal didn't close
- **Won**: Deal closed successfully

---

**For detailed schema:** See `reference.md`
```

---

## Scaling Strategy

### Solo → Team → Enterprise

**Solo Developer (Current):**
- Claude Code + Notion
- 1-5 agents
- Manual orchestration
- Approximate time saved: 2h/week

**Small Team (3-10 people):**
- Claude Code + Notion
- 5-15 agents per process
- Lightweight orchestration (cron jobs)
- Time saved: 10-20h/week team-wide

**Medium Company (10-100 people):**
- Hybrid: Claude Code + Custom Backend
- 20-50 agents across departments
- Event-driven orchestration
- Time saved: 50-200h/week company-wide

**Enterprise (100+ people):**
- Custom Agent Framework
- 100+ specialized agents
- Full orchestration platform
- Learning & optimization layer (Level 4)
- Time saved: 500+ hours/week company-wide

---

## Measuring Success

### Metrics by Autonomy Level

**Level 3 (Current):**
- Agent accuracy: >80% of suggestions accepted
- Time saved per operation: 5-10 minutes
- Adoption rate: >70% of applicable tasks
- User satisfaction: 4+/5

**Level 4 (Target):**
- Agent accuracy: >95% (learning from corrections)
- Autonomous error recovery: >80% of issues
- Cross-session memory: 100% of corrections retained
- Confidence calibration: Accuracy ±5% of stated confidence

### Business Metrics

**Efficiency:**
- Pipeline velocity: Days in Stage reduction
- Update frequency: % of opportunities current (<7 days old)
- Meeting prep time: Minutes saved per meeting

**Quality:**
- Consistency: Standard deviation of Update Log quality
- Follow-through: % of Next Actions completed on time
- Forecast accuracy: Pipeline vs actual revenue

**Adoption:**
- Agent usage rate: % of operations using agents
- User preference: Agent vs manual execution
- Expansion: New agents requested by users

---

## Future Research Areas

### Level 4 → Level 5 Evolution

1. **Multi-Agent Negotiation**
   - Agents debate best course of action
   - Consensus-based decision making

2. **Predictive Pipeline Management**
   - Forecast deal closure probability
   - Suggest proactive interventions

3. **Autonomous Workflow Optimization**
   - Agents identify process inefficiencies
   - Self-modify workflows for improvement

4. **Natural Language Orchestration**
   - "Keep pipeline healthy" → Agent determines and executes all necessary actions
   - Zero manual intervention

---

## References

### Documentation Sources
- **Claude Code Docs:** https://code.claude.com/docs
- **Skills Guide:** https://code.claude.com/docs/en/skills
- **Sub-agents Guide:** https://code.claude.com/docs/en/sub-agents
- **Andrew Ng Agentic Course:** https://www.deeplearning.ai/short-courses/

### Internal Resources
- Process 2.2 Specification: `docs/2.2-opportunity-advancer.md`
- Current Architecture: `docs/arquitetura.md`
- Implementation Roadmap: `docs/cronograma.md`
- CRM Workflow Documentation: Notion workspace

---

**End of Document**
**Version:** 1.0
**Status:** Reference for future scaling
**Next Review:** When transitioning to Level 4 autonomy
