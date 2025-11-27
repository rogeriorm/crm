# Agentic Workflow Map

**Version:** 1.1
**Last Updated:** 2025-11-27
**Status:** Building incrementally as needed

**Changelog:**
- v1.1 (2025-11-27): Added Agent Governance Layer as third perspective
- v1.0 (2025-11-23): Initial version with Business and Technical flows

## Overview

This document maps the current and planned agentic workflow architecture for the CRM project. Three complementary perspectives are presented:

1. **Business Operations Flow** - Customer-facing agents that drive revenue
2. **Technical Enablement Flow** - Meta-agents that improve the system itself
3. **Agent Governance Layer** - Cross-cutting concern for audit, control, validation, and learning

All flows embody the principle: **Works → Fast → Validated → Iterated**

---

## Agent Governance Layer (Cross-Cutting)

### Architecture Principle

Governance is **NOT** a separate agent or standalone service. It's a **cross-cutting concern** that weaves through all agent operations at every phase of SPAR.

```
┌─────────────────────────────────────────────────────────────────┐
│                   AGENT GOVERNANCE LAYER                        │
│         (Cross-cutting: Audit, Control, Validation)            │
└─────────────────────────────────────────────────────────────────┘
                               │
        ┌──────────────────────┼──────────────────┬──────────────┐
        │                      │                  │              │
        ↓                      ↓                  ↓              ↓
   SENSE Phase            PLAN Phase         ACT Phase     REFLECT Phase
        │                      │                  │              │
   ┌────┴─────┐          ┌─────┴──────┐    ┌─────┴──────┐  ┌────┴─────┐
   │ Schema   │          │ Business   │    │ Rate       │  │ Failure  │
   │ Validate │          │ Rule       │    │ Limiting   │  │ Registry │
   │          │          │ Validate   │    │ Audit Log  │  │ Rollback │
   └──────────┘          └────────────┘    │ User Gate  │  │ Learning │
                                           └────────────┘  └──────────┘
```

### Progressive Evolution Model

Governance grows with system complexity:

```
Phase 0: Lightweight Practices (1 agent)
  ├─ Human approval = governance gate
  ├─ Update Log = audit trail
  ├─ CLAUDE.md = schema/rules
  └─ Manual rollback (screenshots)

Phase 1: Shared Conventions (3+ agents)
  ├─ /docs/governance/ folder
  ├─ audit-format.md
  ├─ failure-log.md
  ├─ operational-limits.md
  └─ schema-reference.json

Phase 2: Reusable Subsystem (5+ agents)
  ├─ /.claude/governance/ code
  ├─ audit-logger.js
  ├─ rate-limiter.js
  ├─ schema-validator.js
  └─ rollback-manager.js

Phase 3: Dedicated Service (Orchestration)
  ├─ /services/governance/ APIs
  ├─ audit-service
  ├─ policy-engine (auto-approve rules)
  ├─ monitoring-dashboard
  └─ rollback-service
```

### Current State: Phase 0

**Status:** ✅ Operational (sufficient for current scale)

**Implementation:**
- Human approval gate (all writes require confirmation)
- Update Log field (implicit audit in Notion)
- CLAUDE.md constraints (documented rules)
- SPAR framework (built-in validation)
- ✅ IMPLEMENTED: `docs/governance/failure-log.md` created (Issue #10)

**Key Principle:** "Earn Your Infrastructure"
- Don't build ahead of evidence
- Governance emerges from complexity
- Human approval IS the governance layer at this scale

### Signals to Advance Phases

| Signal | Threshold | Action |
|--------|-----------|--------|
| Same failure 3x | Pattern emerged | Build targeted prevention (#9 schema validation) |
| Recovery > 1 hour | Stakes too high | Build rollback (#8) |
| Approval fatigue | >20 approvals/session | Build auto-approve (#13) |
| Cost surprise | Bill > 2x expected | Build monitoring (#15) |
| 3+ agents operational | Shared conventions needed | Move to Phase 1 |
| 5+ agents operational | Volume justifies infra | Move to Phase 2 |

### GitHub Issues Mapped to Governance

**Tier 1: Core Practices (Now)**
- #10: Failure case registry → `docs/governance/failure-log.md` (15 min)

**Tier 2: Targeted Solutions (When Pattern Emerges)**
- #9: Schema validation → Build after 3rd schema error
- #12: Rate limiting → Build after first runaway operation
- #13: Auto-approval → Build after approval bottleneck

**Tier 3: Infrastructure (At Scale)**
- #8: Audit log → Build when debugging becomes hard
- #15: Monitoring → Build when costs become significant
- #14: Staging DB → Build when production testing too risky
- #11: Key rotation → Implement as periodic process

---

## Business Operations Flow

### Current State (Operational)

```
┌─────────────────────────────────────────────────────────────────┐
│                     CUSTOMER LIFECYCLE                          │
└─────────────────────────────────────────────────────────────────┘

    Opportunity Created
           │
           ↓
    ┌──────────────────┐
    │ opportunity-     │  ✅ OPERATIONAL (Phase 0)
    │ advancer         │
    └──────────────────┘
           │
           │  Process:
           │  1. Search Oportunidades by name (limit: 5 results)
           │  2. Show multiple results if found ⭐ IMPROVED
           │  3. Validate selected result with user ⭐ NEW
           │  4. Log search accuracy to memory ⭐ NEW
           │  5. Fetch opportunity data
           │  6. Analyze interactions
           │  7. Update 6 fields:
           │     - Update Log
           │     - Next Action
           │     - NAction Due
           │     - Biz Funnel Stage
           │     - Status
           │     - AI Recommendation
           │
           ↓
    [11-stage Biz Funnel]
           │
           │  Marketing → Suspect → Prospect → Contato →
           │  Credibilidade → Oferta → Proposta →
           │  Negociação → Fechamento → Relacionamento → done!
           │
           ↓
    ┌──────────────────┐
    │ customer-        │  🚧 GAP (Planned - Phase 2+)
    │ success          │
    └──────────────────┘
           │
           │  Post-sale activities:
           │  - Onboarding tracking
           │  - Health score monitoring
           │  - Renewal/upsell signals
           │  - Churn prevention
           │
           ↓
    Customer Retained/Expanded
```

### Identified Gaps (Business Flow)

1. **Pre-meeting preparation** (Phase 1)
   - Agent: `meeting-prep-briefer`
   - Triggers: 24h before scheduled meeting
   - Output: Context summary, talking points, risk flags

2. **Weekly pipeline digest** (Phase 1)
   - Agent: `pipeline-digest`
   - Triggers: Monday mornings
   - Output: Stalled opportunities, action items, priorities

3. **Follow-up drafting** (Phase 1)
   - Agent: `follow-up-drafter`
   - Triggers: Post-meeting, Next Action detected
   - Output: Email/message draft based on meeting outcome

4. **Customer success management** (Phase 2+)
   - Agent: `customer-success`
   - Triggers: Stage = "Relacionamento"
   - Output: Health monitoring, renewal signals, expansion opportunities

5. **Deal risk analysis** (Phase 2+)
   - Agent: `deal-risk-analyzer`
   - Triggers: High-value opportunities stalling
   - Output: Risk assessment, recommended interventions

---

## Technical Enablement Flow

### Current State (Operational)

```
┌─────────────────────────────────────────────────────────────────┐
│                  SYSTEM IMPROVEMENT CYCLE                       │
└─────────────────────────────────────────────────────────────────┘

    User Feedback/Observation
           │
           ↓
    ┌──────────────────┐
    │ cs-agent-        │  ✅ OPERATIONAL
    │ validator        │
    └──────────────────┘
           │
           │  Validates feedback against:
           │  - Project specifications (PRINCIPLES.md)
           │  - Agent behavior expectations
           │  - Defined workflows
           │  - Creates GitHub issues
           │
           ↓
    GitHub Issue Created (e.g., #2)
           │
           ├─────────────────────────────┐
           │                             │
           ↓                             ↓
    ┌──────────────────┐         ┌──────────────────┐
    │ ai-enablement-   │         │ business-        │
    │ reviewer         │         │ architect        │
    └──────────────────┘         └──────────────────┘
    ✅ OPERATIONAL               ✅ OPERATIONAL
           │                             │
           │  Reviews for:               │  Applies SPAR:
           │  - Token efficiency         │  - Sense: Business context
           │  - Business value           │  - Planning: Minimal solution
           │  - Lean leverage            │  - Acting: Implementation
           │  - Design quality           │  - Reflect: Metrics & iteration
           │  - Iterability              │
           │                             │
           └──────────┬──────────────────┘
                      ↓
    ┌──────────────────────────────────────┐
    │ system-implementer                   │  ✅ OPERATIONAL
    └──────────────────────────────────────┘
           │
           │  Executes:
           │  - Reads current system state
           │  - Plans minimal changes (skills → agents → docs)
           │  - Shows proposed changes (approval gates)
           │  - Implements sequentially
           │  - Validates no regression
           │  - Updates all documentation
           │
           ↓
           Implementation → Testing → Iteration
                      │
                      ↓
           New Agent/Skill Created
                      │
                      ↓
           (cycle repeats with cs-agent-validator)
```

### Identified Gaps (Technical Flow)

1. **Automated testing agent** (Future)
   - Agent: `agent-tester`
   - Triggers: Agent config changes
   - Output: Test results, regression checks

2. **Performance monitoring** (Future)
   - Agent: `performance-monitor`
   - Triggers: Periodic (daily/weekly)
   - Output: Token usage reports, latency analysis, cost optimization

3. **Pattern extraction** (Future)
   - Agent: `pattern-extractor`
   - Triggers: After N successful iterations
   - Output: Reusable patterns, skill candidates, abstraction opportunities

---

## Three-Layer Integration Map

```
┌────────────────────────────────────────────────────────────────┐
│                COMPLETE SYSTEM ARCHITECTURE                    │
└────────────────────────────────────────────────────────────────┘

                    ┌─────────────────────────────┐
                    │  AGENT GOVERNANCE LAYER     │
                    │  (Cross-cutting concern)    │
                    └──────────────┬──────────────┘
                                   │
              ┌────────────────────┼────────────────────┐
              │                    │                    │
              ↓                    ↓                    ↓
    ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐
    │   BUSINESS      │  │   TECHNICAL     │  │   GOVERNANCE    │
    │   OPERATIONS    │  │   ENABLEMENT    │  │   CONTROLS      │
    │                 │  │                 │  │                 │
    │ opportunity-    │  │ cs-agent-       │  │ • Audit logs    │
    │ advancer ✅     │  │ validator ✅    │  │ • Rate limits   │
    │                 │  │                 │  │ • Schema checks │
    │ pipeline-digest │  │ ai-enablement-  │  │ • Rollback      │
    │ 🚧 planned      │  │ reviewer ✅     │  │ • User gates    │
    │                 │  │                 │  │ • Failure log   │
    │ meeting-prep    │  │ business-       │  │                 │
    │ 🚧 planned      │  │ architect ✅    │  │ Phase 0: ✅     │
    │                 │  │                 │  │ Phase 1: 🚧     │
    │ follow-up       │  │ system-         │  │ Phase 2: 🔮     │
    │ 🚧 planned      │  │ implementer ✅  │  │ Phase 3: 🔮     │
    └─────────────────┘  └─────────────────┘  └─────────────────┘
              │                    │                    │
              └────────────────────┴────────────────────┘
                                   │
                                   ↓
                    ┌─────────────────────────────┐
                    │     NOTION DATA LAYER       │
                    │  (Single source of truth)   │
                    └─────────────────────────────┘
```

## Interaction Map: Business ↔ Technical ↔ Governance

```
┌────────────────────────────────────────────────────────────────┐
│                    FEEDBACK LOOPS                              │
└────────────────────────────────────────────────────────────────┘

Business Operations → Technical Enablement:
─────────────────────────────────────────
• opportunity-advancer usage data → ai-enablement-reviewer
  "Token consumption too high? Optimize prompts"

• Stalled opportunities pattern → business-architect
  "Repetitive manual intervention? Design automation"

• User feedback on recommendations → ai-enablement-reviewer
  "Low accuracy in Biz Funnel detection? Refine logic"

Technical Enablement → Business Operations:
─────────────────────────────────────────
• business-architect identifies workflow gap → new agent spec
  "No follow-up tracking? Create follow-up-drafter"

• ai-enablement-reviewer finds redundancy → consolidation
  "Update Log + Notas overlap? Consolidate with #ai tags"

• Performance analysis shows bottleneck → refactoring
  "Multi-fetch slows down? Batch operations"

Governance → Both Flows:
────────────────────────
• Failure log shows pattern → Trigger schema validation (#9)
  "3x same field error → Add validation before update"

• Approval fatigue detected → Trigger auto-approve (#13)
  ">20 approvals in session → Selective auto-approval needed"

• Recovery cost high → Trigger rollback (#8)
  "1 hour to fix mistake → Build rollback capability"

Both Flows → Governance:
────────────────────────
• opportunity-advancer mistake → Log to failure registry
  "Wrong Biz Funnel stage → Document pattern for prevention"

• Multiple agents operational → Advance governance phase
  "3 agents running → Move from Phase 0 to Phase 1"

• High-value operation → Increase governance rigor
  "Critical update → Require 2-step approval"
```

---

## Agent Catalog

### Operational Agents

| Agent | Status | Type | Trigger | Output |
|-------|--------|------|---------|--------|
| **opportunity-advancer** | ✅ Operational | Business | User: "analyze opportunity [Name]" | 6 field updates |
| **cs-agent-validator** | ✅ Operational | Technical | User provides feedback/observations | Validated feedback + GitHub issue |
| **ai-enablement-reviewer** | ✅ Operational | Technical | User: agent/skill completion | Optimization recommendations |
| **business-architect** | ✅ Operational | Technical | User: process design request | SPAR-based architecture |
| **system-implementer** | ✅ Operational | Technical | User: "implement issue #N" or "implement [description]" | Code changes + validation checklist |
| **customer-success** | 🚧 Planned | Business | Stage = Relacionamento | Health monitoring, renewal signals |

### Planned Agents (Prioritized)

| Agent | Priority | Phase | Depends On | Estimated Effort |
|-------|----------|-------|------------|------------------|
| **pipeline-digest** | High | 1 | opportunity-advancer validated | 2-3 hours |
| **meeting-prep-briefer** | High | 1 | opportunity-advancer validated | 2-3 hours |
| **follow-up-drafter** | Medium | 1 | opportunity-advancer validated | 2-3 hours |
| **deal-risk-analyzer** | Medium | 2 | pipeline-digest operational | 3-4 hours |
| **customer-success** | Low | 2+ | Relacionamento stage in use | 4-5 hours |

---

## Workflow Principles

### 1. Build as Needed (Not Ahead)

```
❌ BAD: Design all 10 agents upfront
✅ GOOD: Build opportunity-advancer → validate → identify next gap
```

**Current approach:**
- opportunity-advancer operational → validating with real data
- Identified gap: Update Log redundancy → Issue #2 created
- Next build: TBD based on usage feedback

### 2. Progressive Complexity

```
Phase 0: Single agent (opportunity-advancer) ✅
Phase 1: 2-3 independent agents (pipeline-digest, meeting-prep) 🚧
Phase 2: Agent coordination (follow-up triggered by opportunity-advancer) 🔮
Phase 3: Full orchestration (multi-agent workflows) 🔮
```

**Never skip phases.** Validate each before advancing.

### 3. Two Flows, One Philosophy

Both Business and Technical flows follow:

**SPAR at every level:**
- Sense: Understand context deeply
- Planning: Design minimal viable solution
- Acting: Implement concretely
- Reflect: Measure, learn, iterate

**Lean principles:**
- Works > Perfect
- Fast > Feature-complete
- Validated > Theoretical
- Iterated > Static

---

## Metrics & Success Criteria

### Business Flow Metrics

**opportunity-advancer (current):**
- ✅ Accuracy: % match with Notion AI Process 2.2 (target: >80%)
- ✅ Usage: # opportunities analyzed per week
- ✅ Time savings: Manual update time vs agent time
- 🚧 User satisfaction: Approval rate on recommendations

**customer-success (future):**
- Churn prediction accuracy
- Renewal rate improvement
- Expansion revenue influenced

### Technical Flow Metrics

**cs-agent-validator (current):**
- ✅ Feedback validations completed
- ✅ GitHub issues created from validated feedback
- ✅ Alignment with project specifications maintained
- ✅ Example: Issue #2 (Update Log → Notas consolidation)

**ai-enablement-reviewer (current):**
- ✅ Reviews completed per week
- ✅ Token optimization recommendations implemented
- ✅ Business value increase from optimizations

**business-architect (current):**
- ✅ Process improvements designed
- ✅ Implementation time vs estimate
- ✅ Adoption rate of designed processes

---

## Evolution Strategy

### How Gaps Get Filled

1. **Usage reveals pain point**
   - Example: "I spend 30 min every Monday reviewing pipeline status"

2. **business-architect designs solution**
   - Applies SPAR framework
   - Designs `pipeline-digest` agent
   - Estimates effort: 2-3 hours

3. **Implementation**
   - Build minimal viable version
   - Test with real data
   - Refine based on feedback

4. **ai-enablement-reviewer optimizes**
   - Reviews implementation
   - Suggests token optimizations
   - Ensures business value delivery

5. **Iteration**
   - Gather usage data
   - Identify next gap
   - Repeat cycle

### When NOT to Build

Don't build an agent if:
- ❌ Manual process takes <5 min
- ❌ Only happens once per month
- ❌ No clear business value
- ❌ Existing agent can be extended
- ❌ User can do it better/faster manually

**Exception:** Build if it's a learning opportunity for future high-value automation.

---

## Integration Points

### Data Sources (Current)

1. **Notion Oportunidades Database**
   - Collection ID: `collection://201b1882-308d-4524-8a86-6672d5502299`
   - Accessed via: MCP Notion integration
   - Permissions: Search/Fetch (auto-approve), Write (user approval)

2. **Anotações Pages**
   - Related to opportunities
   - Contains: transcripts, summaries, notes
   - Format: Notion-flavored markdown with tags

### Integration Patterns

**Current (Phase 0):**
```
User → Claude Code → MCP Notion → Notion API → Database
```

**Future (Phase 2+):**
```
Agent A → State → Agent B → State → Agent C
         ↓                ↓
    Notion DB      Notion DB
```

**Key principle:** State management stays in Notion (single source of truth).

---

## Anti-Patterns to Avoid

### 1. Premature Orchestration

```
❌ BAD: Design multi-agent workflow before single agent works
✅ GOOD: Validate opportunity-advancer alone first
```

### 2. Over-Engineering Gaps

```
❌ BAD: "We might need customer-success eventually, let's design it now"
✅ GOOD: "Do we have Relacionamento stage opportunities? No? Don't build yet."
```

### 3. Duplicate Logic

```
❌ BAD: Each agent implements Biz Funnel rules independently
✅ GOOD: crm-data-model skill centralizes rules, agents reference it
```

### 4. Neglecting Meta-Agents

```
❌ BAD: Build 5 business agents without reviewing any
✅ GOOD: Use ai-enablement-reviewer after each agent
```

---

## Next Steps (Immediate)

1. **Validate opportunity-advancer** (Phase 0)
   - Test with 10+ real opportunities
   - Compare outputs with Notion AI
   - Gather user feedback
   - Measure time savings

2. **Resolve Issue #2** (Technical debt)
   - Consolidate Update Log → Notas
   - Implement #ai tagging
   - Update crm-data-model skill
   - Update opportunity-advancer agent

3. **Identify Phase 1 priority** (Business need)
   - Most painful manual task?
   - pipeline-digest vs meeting-prep vs follow-up-drafter
   - Design with business-architect
   - Build & validate

4. **Document patterns** (Knowledge capture)
   - What worked in opportunity-advancer?
   - What failed/required iteration?
   - Extract reusable patterns for future agents

---

## Real-World Example: cs-agent-validator in Action

### Issue #2: Update Log Consolidation

**User Feedback (2025-11-23):**
> "Existe uma redundância entre o campo Update Log e Notas. No caso, acredito que seja melhor, mover o processo de Update Log para o corpo do Notas, registrando o timestamp. No caso, as alterações deveriam ter uma tag #ai para identificar quando são feitas pelo sistema."

**cs-agent-validator Process:**

1. **Validates against PRINCIPLES.md:**
   - ✅ Principle 8 (Strategic Context Logging)
   - ✅ Principle 16 (Iterative Refinement Over Perfection)

2. **Analyzes redundancy:**
   - Confirmed overlap between Update Log and Notas
   - Identified improvement opportunity (timestamps + #ai tag)

3. **Recommends solution:**
   - Consolidate Update Log into Notas/Histórico section
   - Use `### YYYY-MM-DD HH:MM #ai` format
   - Tag #ai for system-generated updates

4. **Creates GitHub Issue #2:**
   - Complete analysis and recommendation
   - Implementation checklist (4 steps, ~1-1.5h)
   - Impact assessment (benefits + risks)

**Outcome:**
- Validated feedback preserved as actionable issue
- Aligned with lean principles (eliminate redundancy)
- Clear implementation path defined
- Ready for future resolution

**Flow:**
```
User Feedback → cs-agent-validator → Issue #2 →
  (future) ai-enablement-reviewer → Implementation → Validation
```

---

## Appendix: Agent Templates

### Business Agent Checklist

When designing a new business operations agent:

- [ ] Clear business value proposition (measured in time/revenue)
- [ ] Defined trigger conditions (when does it run?)
- [ ] Expected output format (what does user get?)
- [ ] Success criteria (how do we know it works?)
- [ ] Iteration plan (how do we improve it?)
- [ ] Integration points (what data sources? what updates?)
- [ ] Human validation gates (when does user approve?)

### Technical Agent Checklist

When designing a new technical enablement agent:

- [ ] Target improvement area (token efficiency? design quality?)
- [ ] Evaluation framework (how does it assess quality?)
- [ ] Output structure (recommendations format?)
- [ ] Action triggers (when does it provide value?)
- [ ] Feedback loop (how do improvements get measured?)

---

## References

- **PRINCIPLES.md** - 20 core principles guiding development
- **phase-0-lean-implementation.md** - Current implementation phase
- **spar-architecture-future-reference.md** - SPAR framework details
- **.claude/agents/** - Agent configurations
- **.claude/skills/** - Shared skill library

---

**Philosophy:** Build the minimum, validate with reality, iterate based on evidence. Both business and technical flows serve the same master: delivering measurable value to users.
