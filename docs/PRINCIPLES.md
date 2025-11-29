# AI Development Principles - CRM Project

This document captures core principles and learnings from building AI-assisted CRM workflow automation using Claude Code + Notion via MCP.

**Last Updated:** 2025-11-26
**Source Sessions:** 2025-11-21, 2025-11-22, 2025-11-26

---

## Core Philosophy

### The Four Priorities (in order)

1. **Works** - Produces useful output
2. **Fast** - <3 hours per agent
3. **Validated** - Tested with real data
4. **Iterated** - Refined based on usage

### What to Avoid

- ❌ Complex orchestration before basics work
- ❌ Over-engineering for edge cases
- ❌ Abstractions before patterns emerge
- ❌ Premature optimization

---

## Architecture Principles

### 1. SPAR Framework for Agents

Every agent follows four phases:

**SENSE (Load Context)**
- Load all necessary data
- Error handling at entry point
- Fail fast with clear user guidance

**PLAN (Analyze & Generate)**
- Strategic analysis of loaded data
- Apply business rules from skills
- Generate proposed changes

**ACT (Present & Update)**
- Show proposed changes to user
- Wait for approval
- Execute approved changes
- Provide concrete JSON examples

**REFLECT (Validate)**
- Check API responses
- Validate data integrity
- Consider downstream effects
- Log what was analyzed

### 2. Skills vs Agents Separation

**Skills provide:**
- Database schemas
- Business rules
- Field definitions
- Advancement signals
- Strategic principles

**Agents execute:**
- Data loading
- Analysis workflow
- User interaction
- API updates

**Critical Rule:** Agents CANNOT invoke skills directly. Skills are conversation-level context, not agent-level tools.

### 3. Prerequisites Documentation

Every agent must document:
- Required skill context
- Expected data sources
- Fallback instructions if context missing

Example:
```markdown
## Prerequisites
**IMPORTANT:** This agent expects the `crm-data-model` skill context
to be available in the conversation.

If skill context is not loaded, reference
`.claude/skills/crm-data-model/SKILL.md` directly.
```

---

## Implementation Principles

### 4. Error Handling is Non-Negotiable

Every external dependency needs failure cases:

**Search Operations:**
- No results found → ask user to verify name or provide URL
- Multiple results → show list and ask user to choose

**Data Loading:**
- No meeting data → inform user to add content first
- Missing required fields → default values + notify user

**API Updates:**
- Check response status
- Validate format errors (especially dates)
- Show error messages to user

**Enforcement (Added 2025-11-29):**
- **SENSE Phase Requirement:** All agents MUST validate external dependencies (MCP tools, APIs) before use
- **Failure Mode:** Agent should fail fast with clear message if required tools unavailable
- **Anti-Pattern:** Proceeding with execution assuming tools exist, leading to cryptic errors
- **Example:** opportunity-advancer checks for `mcp__notion__notion-fetch` availability before attempting data load

See: `docs/governance/failure-log.md` - 2025-11-29 failure entry

### 5. Be Explicit, Never Vague

**Bad (Vague):**
```markdown
Load recent interactions
```

**Good (Explicit):**
```markdown
Load interactions (2-3 most recent):
- First: Check page content for inline tags
- Second: If no inline content, fetch Anotações pages
  - Sort by Last Edited Time (most recent first)
  - Fetch up to 3 pages
  - Skip empty pages
```

**Rule:** If it requires sorting, filtering, or prioritization, specify exactly how.

**Enforcement (Added 2025-11-29):**
- **PLAN Phase Requirement:** All recommendations MUST include What/When/Why
  - **What:** Specific action (not "follow up")
  - **When:** Concrete date or trigger condition
  - **Why:** Business justification tied to current state
- **Anti-Pattern:** Generic advice that leaves user unsure what to do next
- **Examples:**
  - ❌ BAD: "Follow up" (vague, no date, no context)
  - ✅ GOOD: "Agendar reunião de alinhamento para 05/12/2024 após aprovação do budget"

See: `docs/governance/failure-log.md` - 2025-11-29 failure entry

### 6. String Operations Need Exact Specs

**Bad (Ambiguous):**
```markdown
APPEND to existing log
```

**Good (Exact):**
```markdown
APPEND using exact concatenation:
- Format: "{new_entry}\n{existing_update_log}"
- Example result: "22/11: proposta enviada\n18/11: reunião confirmou interesse"
```

**Rule:** Show the exact string operation with example output.

### 7. Multi-Source Analysis Over Summaries

**Don't:**
- Summarize single latest meeting
- Generate generic updates

**Do:**
- Analyze 2-3 recent interactions
- Detect patterns (repeated themes, escalating interest, stalling)
- Extract strategic insights
- Log decisions and advancement signals, not meeting summaries

### 8. Strategic Context Logging

**Log only:**
- Decisions made
- Advancement signals detected
- Stage triggers
- Blockers identified
- Strategic context (explains delays/progress)

**Never log:**
- Generic "had a meeting" entries
- Vague descriptions
- No-value updates

**Format:** `DD/MM: [max 10 words]` - newest first, always APPEND

**Enforcement (Added 2025-11-29):**
- **REFLECT Phase Requirement:** After successful execution, agents SHOULD log analysis to `.claude/memory/[agent-name]-log.jsonl`
- **Format:** JSONL (one JSON object per line) for machine-readable memory
- **Purpose:** Enable pattern detection, quality review, continuous improvement
- **Error Handling:** If logging fails, warn user but don't block main workflow
- **Example:** See `.claude/memory/opportunity-analysis-log.jsonl` for schema

See: `docs/governance/failure-log.md` - 2025-11-29 failure entry

---

## Documentation Principles

### 9. Lean Documentation

**Priorities:**
1. What you need to DO (actionable)
2. What you need to KNOW (essential context)
3. What to AVOID (common errors)

**Avoid:**
- Long prose explanations
- Theoretical background
- "Nice to have" information
- Redundant sections

**Prefer:**
- Tables over paragraphs
- Examples over descriptions
- Exact formats over general guidelines

**Target:** 150-200 lines, not 500+

### 10. Session Metadata Documentation

After each significant session, document:
- Token usage statistics
- Files changed (insertions/deletions)
- Key accomplishments
- Critical learnings
- Architectural decisions
- Git status (commit hash, push status)
- Next steps

**Purpose:** Enable continuity across sessions, track agentic journey

**Format:** Follow `docs/session-YYYY-MM-DD-*.md` template

---

## Validation Principles

### 11. Bottom-Up Testing Strategy

Test in layers:
1. **Tools** - Verify MCP tools work (search, fetch, update)
2. **Data Model** - Validate field names, types, formats
3. **Agent Logic** - Test SENSE/PLAN/ACT/REFLECT phases
4. **End-to-End** - Full workflow with real data

**Rule:** Validate foundation before building on it.

### 12. Always Validate Field Names Against Source

**Don't assume from specs.**

**Do:**
1. Fetch actual data from Notion
2. Inspect property names in response
3. Use exact names (case-sensitive)
4. Document any discrepancies

**Examples of mismatches found:**
- Spec: "Task" → Actual: "Nome Oportunidade"
- Spec: "Next Action Date" → Actual: "NAction Due"
- Spec: "Offline Notes" → Actual: Field deleted

### 13. Test with Real Data, Not Synthetic

**Prefer:**
- Real opportunities from production database
- Actual meeting transcripts
- Historical data patterns

**If using test data:**
- Use `#TEST` prefix for clean separation
- Mirror real data structure exactly
- Clean up after validation

---

## Collaboration Principles

### 14. User Control First

**Always:**
- Present proposed changes before executing
- Wait for explicit approval
- Show current vs proposed values
- Explain reasoning for changes

**Plan mode:**
- Use for complex multi-step tasks
- Show incremental progress
- Ask before implementing fixes

**Enforcement (Added 2025-11-29):**
- **REFLECT Phase Requirement:** Agents MUST validate user control before completion
- **Checkpoint Questions:**
  - Did user see complete before/after for ALL modifications?
  - Did user explicitly approve EVERY change?
  - If user rejected: did agent stop and ask for guidance?
  - Were any assumptions made about user intent?
- **Anti-Pattern:** Auto-approving changes, proceeding without explicit confirmation

See: `docs/governance/failure-log.md` - 2025-11-29 failure entry

### 15. Token Efficiency Matters

**Monitor:**
- Current usage vs budget
- Cost per operation
- Documentation token cost

**Optimize:**
- Use lean documentation format
- Avoid redundant context
- Reuse proven structures

**Target:** Stay under 70% of token budget for complex sessions

### 16. Iterative Refinement Over Perfection

**Ship:**
- Working implementation first
- Lean documentation
- Minimal viable feature set

**Then:**
- Gather real usage data
- Identify actual pain points
- Refine based on evidence

**Don't:**
- Add features before validation
- Over-engineer for hypothetical scenarios
- Optimize prematurely

---

## Technical Constraints

### 17. MCP-Specific Rules

**Auto-approved tools:**
- `mcp__notion__notion-search`
- `mcp__notion__notion-fetch`

**Require approval:**
- `mcp__notion__notion-update-page`
- `mcp__notion__notion-create-pages`
- All write operations

**Date property format:**
```json
{
  "date:NAction Due:start": "YYYY-MM-DD",
  "date:NAction Due:is_datetime": 0
}
```

### 18. Notion Database Constraints

**Oportunidades Database:**
- Collection ID: `collection://201b1882-308d-4524-8a86-6672d5502299`
- 11-stage Biz Funnel (cannot skip stages)
- Update Log must APPEND, never replace
- Formulas are read-only (don't attempt to update)

**Biz Funnel Flow:**
Marketing → Suspect → Prospect → Contato → Credibilidade → Oferta → Proposta → Negociação → Fechamento → Relacionamento → done!

**Rule:** Never skip stages (e.g., Credibilidade → Proposta is INVALID)

---

## Git Principles

### 19. Commit Messages

**Format:**
```
type: Brief summary (50 chars max)

Detailed explanation of changes.

## Changes
- Bullet list of specific changes
- Organized by file/component

## Impact/Context
- Why these changes matter
- Testing status

🤖 Generated with [Claude Code](https://claude.com/claude-code)

Co-Authored-By: Claude <noreply@anthropic.com>
```

**Types:** feat, fix, refactor, docs, test, chore

### 20. Commit and Push Protocol

**Always:**
1. Show git status before committing
2. Review changes with user
3. Stage only relevant files
4. Use descriptive commit message
5. Push to origin/main after commit
6. Confirm push success

**Never:**
- Commit unreviewed changes
- Mix unrelated changes in single commit
- Skip push step
- Commit without session documentation

---

## Quality Metrics

### Session Success Criteria

**Excellent (5/5):**
- Clear objective achieved
- All deliverables complete
- Token usage <70%
- Learnings documented
- Changes committed and pushed
- Ready for next phase

**Good (4/5):**
- Main objective achieved
- Some refinement needed
- Token usage <85%
- Key learnings captured

**Needs Improvement (3/5):**
- Objective partially achieved
- Significant gaps remain
- Token usage >85%
- Limited documentation

### Code Quality Indicators

**Good:**
- Comprehensive error handling
- Explicit instructions (no ambiguity)
- Exact string/date formats specified
- Prerequisites documented
- Examples included

**Needs Work:**
- Missing error cases
- Vague instructions
- Ambiguous formatting
- Undocumented dependencies
- No examples

---

## Phase-Specific Principles

### Phase 0: Validation
- Mirror existing Notion AI process exactly
- Validate with real data (3-5 opportunities)
- Target: >80% match with Notion AI output
- Document discrepancies
- Iterate based on actual failures

### Future Phases (1-4)
- Build on validated foundation
- Reuse proven patterns
- Maintain token efficiency
- Document learnings continuously

---

## Anti-Patterns to Avoid

### ❌ Premature Abstraction
Don't create helpers/utilities for one-time operations. Three similar lines of code is better than a premature abstraction.

### ❌ Over-Engineering
Don't add features, refactor code, or make "improvements" beyond what was asked. A bug fix doesn't need surrounding code cleaned up.

### ❌ Backwards Compatibility Hacks
Don't use renaming tricks, re-exports, or `// removed` comments. If something is unused, delete it completely.

### ❌ Hypothetical Requirements
Don't design for future scenarios. Build exactly what's needed now.

### ❌ Generic Error Handling
Don't add error handling for scenarios that can't happen. Only validate at system boundaries (user input, external APIs).

### ❌ Excessive Documentation
Don't add docstrings, comments, or type annotations to code you didn't change. Only add comments where logic isn't self-evident.

---

## Success Patterns

### ✅ Skills-First Architecture
Define business rules in skills, reference from agents. Clean separation of concerns.

### ✅ Multi-Source Pattern Recognition
Analyze 2-3 interactions for patterns, not just latest meeting summary.

### ✅ Strategic Context Logging
Log WHY, not just WHAT. Explain delays, blockers, advancement signals.

### ✅ Bottom-Up Validation
Test foundation first (tools → data model → agent → end-to-end).

### ✅ Lean Documentation with Tables
Use tables for reference data. Max 150-200 lines. Actionable over theoretical.

### ✅ Session Metadata Capture
Document token usage, changes, learnings, decisions after every significant session.

### ✅ Explicit Over Implicit
Specify exact sorting, filtering, concatenation. Show examples.

### ✅ Error Handling at Entry Points
Fail fast in SENSE phase with clear user guidance.

---

## Data Consistency & Governance

### 21. Data-Only Analysis - Never Enrich with External Knowledge

**Rule:** Agents must work EXCLUSIVELY with data present in Notion database (via MCP fetch). NEVER add context, descriptions, or information from model's general knowledge.

**Why this matters:**
- External knowledge may be incorrect (proven by Clarius hallucination case)
- Breaks user trust ("it overstated comments that do not exist in my database")
- Violates "Always Validate Against Source" principle
- Creates false confidence in analysis

**Examples of VIOLATIONS:**
- ❌ User mentions "Clarius" → Agent adds "Clarius is an ultrasound company"
- ❌ Opportunity field shows "Microsoft" → Agent adds "a technology giant based in..."
- ❌ Update Log says "waiting contract" → Agent adds "typical for enterprise sales cycles of 6-12 months"
- ❌ Next Action: "Present panel using cases Clarius, Bayer, Indorama" → Agent describes what each company does

**Examples of CORRECT behavior:**
- ✅ User mentions "Clarius" → Agent treats it as opaque string from user's context
- ✅ Opportunity shows "Microsoft" → Agent uses name exactly as appears, no elaboration
- ✅ Update Log says "waiting contract" → Agent analyzes timeline/patterns in THIS opportunity's data only
- ✅ Next Action lists companies → Agent acknowledges the action without describing companies

**Allowed additions:**
- ✅ Patterns detected ACROSS multiple interactions in Notion (e.g., "pricing mentioned in 3 interactions")
- ✅ Inferences FROM the data (e.g., "no response for 21 days suggests stalling")
- ✅ Business rule applications FROM skill definitions (e.g., "pricing discussion signals Oferta stage")

**Not allowed additions:**
- ❌ Industry context not in database
- ❌ Company descriptions not in database
- ❌ Generic business knowledge not tied to specific Notion data
- ❌ Assumptions about entities based on their names

**How to handle missing information:**
- State explicitly: "Field X is empty" or "No information about Y in database"
- Do NOT infer, assume, or fill gaps with general knowledge
- Suggest user adds information to Notion if needed

### 22. Post-Processing Validation Pattern

**Rule:** When external systems (MCP, APIs) return data, apply relevance filtering before presentation to user.

**Why:** External search systems (like Notion AI search) may return semantically related but irrelevant results.

**Pattern:**
1. Receive raw results from external system
2. Apply relevance filter (e.g., substring matching, schema validation)
3. Log both raw count and filtered count for auditability
4. Present only filtered results to user
5. Clear error message if all results filtered out

**Example (already implemented):**
```
Opportunity Search Protocol:
- MCP returns 8 results for "sustentec"
- Post-filter: Keep only results where name contains "sustentec"
- Result: 1 relevant opportunity (7 filtered out)
- Memory log: "8 raw results filtered to 1"
```

**When to apply:**
- MCP notion-search results
- External API responses
- Any AI-powered search/retrieval
- Data imports from external sources

### 23. Multi-Layer Validation Strategy

**Rule:** Apply validation at three checkpoints in agent workflow.

**Three Validation Layers:**

**LAYER 1: INPUT VALIDATION** (SENSE phase)
- Schema validation (field names, types exist?)
- Required data check (necessary fields populated?)
- Error handling (clear messages for missing data)
- Tool availability check (MCP tools accessible?)

**LAYER 2: PROCESS VALIDATION** (PLAN phase)
- Business rules enforcement (Biz Funnel transitions valid?)
- Data source validation (information comes from Notion?)
- Multi-source correlation (patterns across interactions?)
- Anti-hallucination checks (no external knowledge added?)

**LAYER 3: OUTPUT VALIDATION** (before ACT phase)
- Consistency checks (field co-dependencies respected?)
- Quality checks (format compliance, length constraints?)
- Completeness checks (all required fields generated?)
- Governance checks (audit trail, user approval required?)

**Example workflow:**
```
INPUT → [Validate schema] → PROCESS → [Validate business rules] →
OUTPUT → [Validate consistency] → PRESENT → User Approval → ACT
```

### 24. Progressive Trust Model

**Rule:** Trust level varies based on agent history, data criticality, and user context. Adjust validation strictness accordingly.

**Trust Matrix:**

| Criticality | Confidence | Strategy |
|-------------|-----------|----------|
| High + Low | Strict validation + user approval | Biz Funnel changes |
| High + High | Moderate validation + warning flags | Next Action Date calc |
| Low + Low | Light validation + user can override | Formatting suggestions |
| Low + High | Trust-first + monitor for issues | Sorting, metadata |

**Progressive approach:**
- **Phase 0:** User validates everything manually
- **Phase 1:** System validates obvious errors, user validates strategic decisions
- **Phase 2:** System validates majority, user validates critical changes
- **Phase 3:** System auto-validates with confidence scores, user audits periodically

**Current state:** Phase 0/1 transition (opportunity-advancer operational)

### 25. Validation as Documentation

**Rule:** Validation logic IS documentation of business rules, data quality expectations, and system boundaries.

**Principle:**
- Every validation rule documents an expectation
- Failed validations reveal edge cases
- Validation evolution shows system maturity
- Validation logs = living specification

**Examples:**
```
Validation: "Status = 'Waiting Feedback' requires NAction Due to exist"
→ Documents: Status and NAction Due are co-dependent

Validation: "NAction Due must be >= today"
→ Documents: No backdated action dates allowed

Validation: "Update Log entry ≤ 10 words"
→ Documents: Log entries must be concise
```

**Benefits:**
- New developers understand constraints by reading validation code
- Business rules stay synchronized with implementation
- Validation failures guide debugging
- Documentation never drifts from reality

---

## Quick Reference Checklist

When building a new agent:

- [ ] SPAR structure (SENSE/PLAN/ACT/REFLECT)
- [ ] Prerequisites section (skill dependencies)
- [ ] Error handling (3+ scenarios)
- [ ] Explicit instructions (sorting, filtering)
- [ ] Exact string formats (with examples)
- [ ] Date property format documented
- [ ] User approval flow (show before update)
- [ ] API validation in REFLECT
- [ ] Testing checklist created
- [ ] Token usage monitored

When documenting:

- [ ] Tables over prose
- [ ] Examples over descriptions
- [ ] Actionable over theoretical
- [ ] Target: 150-200 lines max
- [ ] Session metadata captured
- [ ] Git commit with full context
- [ ] Pushed to origin/main

---

## Version History

**v1.1** (2025-11-26)
- Added "Data Consistency & Governance" section
- Documented Principles 21-25
- Addresses hallucination prevention (Clarius case)
- Formalizes post-processing validation pattern
- Establishes multi-layer validation strategy
- Implements: Issue #6 (Phase 1 Foundation)

**v1.0** (2025-11-22)
- Initial principles document
- Captured learnings from sessions 2025-11-21 and 2025-11-22
- 20 core principles documented
- Success patterns and anti-patterns identified

---

## Future Evolution

This document will evolve as we:
- Complete Phase 0 validation
- Build additional agents (Phases 1-4)
- Discover new patterns
- Identify new constraints

**Update frequency:** After each major phase completion

**Review:** Every 5 sessions or monthly (whichever comes first)

---

**Remember:** These principles emerged from real implementation experience, not theory. Trust them, but update them when reality provides new evidence.
