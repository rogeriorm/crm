# AI Development Principles - CRM Project

This document captures core principles and learnings from building AI-assisted CRM workflow automation using Claude Code + Notion via MCP.

**Last Updated:** 2025-11-22
**Source Sessions:** 2025-11-21, 2025-11-22

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
