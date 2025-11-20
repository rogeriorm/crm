# Two Alternative Architectural Plans for CRM Agent System

**Date:** 2025-11-20
**Status:** Proposal for Review
**Based on:** Claude Code course learnings, Skills/Sub-agents documentation, and CRM project context

---

## PLAN A: "Skills-First Progressive Architecture"
### Philosophy: Start Simple, Scale Organically

**Core Strategy:** Build reusable skills first, promote to sub-agents only when complexity demands isolation.

### Phase 0 Implementation (Weeks 1-2)

1. **Initialize with `/init`** → Create baseline CLAUDE.md
2. **Document CRM data model** in CLAUDE.md (Oportunidades schema, Biz Funnel stages, business rules)
3. **Create first skill:** `crm-data-model/`
   - SKILL.md with database structure, properties, relationships
   - Auto-loads whenever working with opportunities
4. **Build Process 2.2 in main thread** (NOT sub-agent yet)
   - Use natural conversation: "Analyze this opportunity and suggest next actions"
   - Skills provide context automatically
   - Validate outputs against Notion AI
5. **Extract patterns** into skills as they emerge

### Phase 1 Evolution (Weeks 3-4)

1. **Create `pipeline-digest/` skill**
   - Template-based report generation
   - Metrics calculations in reference.md
2. **Add `funnel-analysis/` skill**
   - Stage transition logic extracted from Process 2.2 learnings
3. **Still working in main thread** with skill support

### Phase 2+ Scaling (When Needed)

1. **Promote to sub-agents ONLY if:**
   - Tool restrictions needed (prevent accidental Notion updates)
   - Complex state across multiple conversations
   - Team members need isolated expertise
2. **Create sub-agents for:**
   - `opportunity-advancer` (restricted write access)
   - `meeting-prep` (read-only research)
   - `follow-up-drafter` (template expansion)

### Structure Example (Plan A)

```
.claude/
├── CLAUDE.md                            # CRM data model, business rules
├── skills/
│   ├── crm-data-model/
│   │   ├── SKILL.md                     # Auto-loads for opportunity context
│   │   └── reference.md                 # Database schema details
│   ├── funnel-analysis/
│   │   └── SKILL.md                     # Stage transition logic
│   └── pipeline-metrics/
│       └── SKILL.md                     # Health score calculations
└── agents/                              # Created later when needed
    └── opportunity-advancer.md          # Only if tool restrictions required
```

### Pros

- ✅ **Fastest to value** (no sub-agent config initially)
- ✅ **Skills evolve naturally** from real usage patterns
- ✅ **Lower cognitive overhead** (work in familiar main thread)
- ✅ **Easy iteration** (edit SKILL.md files directly)
- ✅ **Automatic activation** (skills load based on context)

### Cons

- ⚠️ **Less isolation** (all operations in main context)
- ⚠️ **Tool access not restricted** until sub-agents added
- ⚠️ **May need refactoring** when complexity grows
- ⚠️ **No conversation state** per process

### Best For

- ✅ Your stated priority: "short-term improvements and validation"
- ✅ Rapid prototyping and learning phase
- ✅ Solo developer initially
- ✅ Read-heavy operations (analysis, reporting)

---

## PLAN B: "Sub-Agent Isolation Architecture"
### Philosophy: Build Boundaries First, Share Knowledge via Skills

**Core Strategy:** Each CRM process = dedicated sub-agent with minimal tools. Skills provide shared knowledge base.

### Phase 0 Implementation (Weeks 1-2)

1. **Initialize with `/init`** → Create baseline CLAUDE.md
2. **Create skill foundation:**
   ```
   .claude/skills/
   ├── crm-data-model/SKILL.md          # Database schema
   ├── biz-funnel-stages/SKILL.md       # Stage definitions
   └── email-templates/SKILL.md         # Communication patterns
   ```
3. **Create `opportunity-advancer` sub-agent immediately:**
   ```yaml
   name: opportunity-advancer
   description: "Analyze opportunities and recommend 5 fields update. Use PROACTIVELY when user mentions updating opportunity progress."
   tools: [mcp__notion__notion-search, mcp__notion__notion-fetch]
   model: sonnet
   permissionMode: ask
   ```
4. **Test isolation:** Sub-agent cannot accidentally update Notion (tools restricted)
5. **Compare outputs** with Notion AI Process 2.2

### Phase 1 Evolution (Weeks 3-4)

1. **Create `pipeline-digest` sub-agent:**
   - Read-only tools (search, fetch)
   - Uses `crm-data-model` skill for context
   - Generates weekly report
2. **Skills act as shared knowledge:**
   - All sub-agents access same data model
   - Single source of truth for business rules

### Phase 2+ Scaling

1. **Add focused sub-agents:**
   ```
   .claude/agents/
   ├── opportunity-advancer.md          # DONE (Phase 0)
   ├── pipeline-digest.md               # DONE (Phase 1)
   ├── meeting-prep.md                  # Briefing generation
   ├── opportunity-creator.md           # New opp setup
   └── follow-up-drafter.md             # Email composition
   ```
2. **Skills library grows:**
   - Email templates
   - SPIN questions
   - Proposal templates

### Structure Example (Plan B)

```
.claude/
├── CLAUDE.md                            # High-level project context
├── skills/
│   ├── crm-data-model/
│   │   └── SKILL.md                     # Shared across all agents
│   ├── biz-funnel-stages/
│   │   └── SKILL.md                     # Business rules
│   └── email-templates/
│       └── SKILL.md                     # Communication patterns
└── agents/
    ├── opportunity-advancer.md          # Phase 0: restricted tools
    ├── pipeline-digest.md               # Phase 1: read-only
    ├── meeting-prep.md                  # Phase 2: research
    ├── opportunity-creator.md           # Phase 2: create ops
    └── follow-up-drafter.md             # Phase 2: email drafts
```

### Pros

- ✅ **Clear boundaries from day 1** (safety by design)
- ✅ **Tool restrictions prevent mistakes** (e.g., accidental Notion writes)
- ✅ **Easy to understand** "which agent does what"
- ✅ **Resume-able conversations** per agent (maintain state)
- ✅ **Team collaboration** (different agents, different team members)
- ✅ **Isolated context windows** (focused expertise)

### Cons

- ⚠️ **More upfront configuration** (each sub-agent needs setup)
- ⚠️ **Context switching** between agents
- ⚠️ **May feel over-engineered** for simple tasks initially
- ⚠️ **Two places to maintain** (skills + sub-agents)
- ⚠️ **Manual invocation** or careful description writing for proactive use

### Best For

- ✅ Safety boundaries and clear separation of concerns from day 1
- ✅ Team environments (multiple contributors)
- ✅ Write operations that need approval gates
- ✅ Complex processes requiring conversation state

---

## SKILLS vs SUB-AGENTS: When to Use Each

### Skills (Automatic, Shared Knowledge)

| Use When | Example |
|----------|---------|
| Reusable reference material | CRM data model, schema definitions |
| Consistent procedures | Email templates, report formats |
| Contextual knowledge | Biz Funnel stages, business rules |
| Read-only operations | Documentation, patterns |
| Auto-activation desired | "When user mentions pipeline, load metrics skill" |

**Key Characteristics:**
- Stateless (no conversation memory)
- Automatic activation based on description
- Shared across all agents and main thread
- Efficient (progressive loading of supporting files)

### Sub-Agents (Focused, Isolated Expertise)

| Use When | Example |
|----------|---------|
| Tool restrictions needed | Limit Notion write operations |
| Complex multi-step workflows | Process 2.2 analysis with state |
| Conversation state required | Resume analysis across sessions |
| Team member specialization | Different agents, different owners |
| Security boundaries | Separate sensitive operations |

**Key Characteristics:**
- Stateful (maintain conversation history)
- Isolated context window (own memory)
- Manual invocation or proactive via description
- Can resume previous conversations

---

## DECISION FRAMEWORK

### Choose Plan A (Skills-First) if:

- ✅ You want **fastest path to Phase 0 validation** (2-3 hours)
- ✅ **Iteration speed > safety boundaries** initially
- ✅ You're comfortable working in main thread
- ✅ Process 2.2 mirror is primarily **research/analysis** (not writes)
- ✅ Solo developer learning patterns
- ✅ Want automatic activation of context

### Choose Plan B (Sub-Agent Isolation) if:

- ✅ You want **tool restrictions from day 1** (prevent accidental updates)
- ✅ **Clear boundaries** feel more organized to you
- ✅ You plan to have **team members contribute** different agents
- ✅ You value **resume-able conversation state** per process
- ✅ Write operations need approval gates
- ✅ Complex processes benefit from isolated context

---

## RECOMMENDED APPROACH: **Plan A with Path to B**

### Hybrid Strategy

**Rationale:**
1. Your stated priority: "short-term improvements and validation"
2. Phase 0 goal: Validate hybrid approach quickly (2-3 hours)
3. Plan A gets you to validation faster
4. You can promote main thread patterns to sub-agents later when:
   - Tool restrictions become necessary
   - Complexity justifies isolation
   - Patterns stabilize and are ready for team collaboration

### Execution Timeline

**Week 1: Skills + Main Thread (Plan A)**
```bash
/init                                    # Create CLAUDE.md
# Document CRM data model
# Create crm-data-model/ skill
# Test Process 2.2 in main thread
```

**Week 2: Validate & Decide**
- Compare outputs: Main thread vs Notion AI Process 2.2
- If validation succeeds AND tool restrictions needed → Create `opportunity-advancer` sub-agent
- If validation succeeds AND simple is fine → Continue with skills

**Week 3+: Incremental Evolution**
- Continue with Plan A for **new** agents (e.g., pipeline-digest skill)
- Promote to sub-agents **only when** complexity/isolation justified
- Extract reusable patterns into skills as they emerge

### Migration Path (A → B)

When you decide a skill should become a sub-agent:

1. **Keep the skill** (shared knowledge base)
2. **Create sub-agent** that references the skill
3. **Restrict tools** in sub-agent config
4. **Test in isolation** before full adoption

Example:
```
.claude/
├── skills/
│   └── crm-data-model/SKILL.md          # Stays (shared knowledge)
└── agents/
    └── opportunity-advancer.md          # New (isolated execution)
        ↓ references skill above
```

---

## IMPLEMENTATION CHECKLIST

### Plan A: Skills-First Approach

**Phase 0 (Week 1):**
- [ ] Run `/init` in CRM project directory
- [ ] Document in CLAUDE.md:
  - [ ] Oportunidades database schema
  - [ ] Biz Funnel stages (11 stages)
  - [ ] Process 2.2 business rules
  - [ ] Key properties (Task, Status, Priority, Next Action, etc.)
- [ ] Create `.claude/skills/crm-data-model/`
  - [ ] SKILL.md with database structure
  - [ ] reference.md with detailed schema
- [ ] Test Process 2.2 logic in main thread:
  - [ ] "Analyze opportunity [Name] and suggest 5 field updates"
  - [ ] Validate against Notion AI outputs
  - [ ] Test with 3-5 real opportunities
- [ ] Document learnings and patterns

**Phase 0 Decision Gate:**
- [ ] Outputs match Notion AI ≥80% of the time?
- [ ] Tool restrictions needed? (If YES → create sub-agent)
- [ ] Continue with skills? (If YES → proceed to Phase 1)

**Phase 1 (Week 2-3):**
- [ ] Create `.claude/skills/pipeline-digest/`
  - [ ] SKILL.md with report generation logic
  - [ ] templates/digest-template.md
  - [ ] reference.md with metrics definitions
- [ ] Test pipeline digest in main thread
- [ ] Iterate on format based on real usage
- [ ] Extract more patterns into skills

### Plan B: Sub-Agent Isolation Approach

**Phase 0 (Week 1):**
- [ ] Run `/init` in CRM project directory
- [ ] Create skill foundation:
  - [ ] `.claude/skills/crm-data-model/SKILL.md`
  - [ ] `.claude/skills/biz-funnel-stages/SKILL.md`
  - [ ] `.claude/skills/email-templates/SKILL.md`
- [ ] Create `.claude/agents/opportunity-advancer.md`:
  ```yaml
  name: opportunity-advancer
  description: "Analyze opportunities and recommend 5 fields update. Use PROACTIVELY when user mentions updating opportunity progress after meetings."
  tools:
    - mcp__notion__notion-search
    - mcp__notion__notion-fetch
  model: sonnet
  permissionMode: ask
  ```
- [ ] Test sub-agent isolation:
  - [ ] Verify tool restrictions work
  - [ ] Cannot accidentally update Notion
  - [ ] Skills load automatically for context
- [ ] Compare outputs with Notion AI Process 2.2
- [ ] Test with 3-5 real opportunities

**Phase 1 (Week 2-3):**
- [ ] Create `.claude/agents/pipeline-digest.md`
  - [ ] Read-only tools (search, fetch)
  - [ ] Uses crm-data-model skill
  - [ ] Generates structured report
- [ ] Test digest generation
- [ ] Iterate on sub-agent prompt and output format

---

## KEY CONCEPTS FROM CLAUDE CODE TRAINING

### CLAUDE.md Best Practices
- **Project CLAUDE.md**: Shared with team, committed to git
- **Local CLAUDE.md** (`claude.local.md`): Personal preferences, gitignored
- **What to include**: Mission, key facts, output preferences, constraints
- **What NOT to include**: Full docs (use progressive disclosure via skills)

### Progressive Context Loading
- Skills load supporting files on-demand (not all at once)
- Use `reference.md` for detailed content
- SKILL.md stays concise for discovery

### Sub-Agent Design Principles
- **Single responsibility**: One focused task per agent
- **Minimal tools**: Grant only necessary permissions
- **Detailed prompts**: Include examples and constraints
- **Resume-able**: Use stored transcripts for continuation
- **Proactive activation**: Use phrases like "Use PROACTIVELY when..."

### Skills Design Principles
- **Focused scope**: One capability per skill (not broad domains)
- **Discovery optimization**: Write specific descriptions with key terminology
- **Automatic activation**: Based on contextual relevance
- **Team distribution**: Share via git or package in plugins

---

## NEXT IMMEDIATE ACTIONS

### If Choosing Plan A:
1. Run `/init` in `/Users/rogeriomoreira/code/crm/`
2. Document Oportunidades schema in CLAUDE.md
3. Create first skill: `crm-data-model/`
4. Test Process 2.2 in main thread: "Analyze opportunity [Name]"
5. Validate outputs against Notion AI

**Estimated time to first validation:** 2-3 hours

### If Choosing Plan B:
1. Run `/init` in `/Users/rogeriomoreira/code/crm/`
2. Create skill foundation (3 skills)
3. Create `opportunity-advancer` sub-agent with restricted tools
4. Test sub-agent: invoke and compare with Notion AI
5. Validate isolation (cannot write to Notion accidentally)

**Estimated time to first validation:** 3-4 hours

---

## APPENDIX: Key Learning Resources

### From Research:
- **Notion Repository:** Claude Code course notes with transcripts
- **Sub-agents Docs:** https://code.claude.com/docs/en/sub-agents
- **Skills Docs:** https://code.claude.com/docs/en/skills
- **Claude Skills Blog:** https://www.claude.com/blog/skills
- **Gamma Tutorial:** Iwo Szapar's mini-tutorial approach

### Core Concepts Learned:
- **Agentic search** over indexing (tools-based exploration)
- **Progressive context loading** (on-demand information)
- **Memory via CLAUDE.md** (project/local/global scopes)
- **Plan-first workflow** (Shift+Tab twice for Plan Mode)
- **Skills vs Sub-agents** (automatic vs manual, shared vs isolated)
- **MCP integration** (extend capabilities via protocol servers)

---

**End of Document**
**Version:** 1.0
**Next Review:** After choosing and implementing one approach
