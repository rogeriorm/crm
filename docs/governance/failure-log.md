# Agent Failure Log

**Purpose:** Track agent failures to identify patterns and inform infrastructure decisions.

## How to Use This Log

When an agent makes a mistake:
1. Copy the template below
2. Fill in all fields honestly
3. Add entry under ## Log section (newest first)
4. Commit to git with context

## Entry Template

### [YYYY-MM-DD] [Agent Name] - [Brief Description]

**What Happened:**
[Clear description of the failure]

**Root Cause:**
[Why it happened - be specific]

**Impact:**
- Severity: [Low/Medium/High]
- Data corrupted: [Yes/No]
- User intervention required: [Yes/No]
- Time to recover: [X minutes]

**Prevention:**
[What would prevent this from happening again]

**Related Issue:**
[GitHub issue number if fix is tracked, or "None"]

**Patterns:**
[If this connects to other failures, note the pattern]

---

## Signals to Build Infrastructure

Track these thresholds:
- **Same failure 3x** → Build targeted prevention (schema validation, rate limiting, etc.)
- **Recovery > 1 hour** → Build rollback capability (#8)
- **Approval fatigue (>20/session)** → Build selective auto-approve (#13)
- **Cost surprise** → Build usage monitoring (#15)

## Log

### 2025-11-29 Slash Command Syntax Error - Incorrect Argument Template

**What Happened:**
Implemented `/analyze-opportunity` slash command but used incorrect argument syntax `{{ARG}}` instead of `$ARGUMENTS`. Command not recognized by Claude Code, returned "Unknown slash command" error.

**Root Cause:**
Unfamiliarity with Claude Code slash command syntax specification. Assumed template syntax similar to other systems (e.g., Jinja2, Mustache) but Claude Code uses shell-style variable syntax `$ARGUMENTS` or `$1`, `$2`, etc.

**Impact:**
- Severity: Medium
- Data corrupted: No
- User intervention required: Yes (command non-functional)
- Time to recover: 5 minutes (SAC agent identified issue, quick fix)

**Prevention:**
- Corrected to use `$ARGUMENTS` per official documentation
- Added frontmatter with `description` and `argument-hint` for better UX
- SAC agent effectively diagnosed root cause through spec validation
- Established pattern: always reference official tooling docs for syntax

**Solution Implemented:**
- Replaced `{{ARG}}` with `$ARGUMENTS` throughout command file
- Added YAML frontmatter for discoverability
- Tested with `/help` to verify registration

**Related Issue:**
#18 - Implement slash command for deterministic agent trigger

**Patterns:**
First use of slash commands in project. Lesson learned: Claude Code uses shell-style syntax, not template engine syntax. SAC agent validation against official docs caught this quickly.

---

### 2025-11-29 Auto-Trigger Inconsistency & MCP Isolation

**What Happened:**
Auto-trigger worked for Musashi opportunity (2025-11-29 10:07:32Z) but failed for Ferreira Costa (same day). When user manually invoked agent via "using opportunity advancer", execution failed with "MCP tool not available" error.

**Root Cause:**
1. **Auto-trigger is unreliable:** Agent `description` field is LLM hint, not deterministic hook. LLM interpretation varies across sessions.
2. **MCP tools unavailable in subagents:** When agent invoked via Task tool, executes in isolated context that does NOT inherit MCP server connections from main conversation.
3. **Architectural constraint:** MCP tools are conversation-level resources. Subagent isolation prevents access.

**Impact:**
- Severity: Medium
- Data corrupted: No
- User intervention required: Yes (confusion about why same command works/fails)
- Time to recover: 15 minutes (investigation + workaround)

**Prevention:**
- Created `/analyze-opportunity [Name]` slash command for deterministic trigger
- Slash command executes in main context (MCP access guaranteed)
- Updated agent description to reference slash command as primary invocation method
- Added Principle 27 (Slash Commands for Deterministic Triggers) to PRINCIPLES.md
- Updated memory log schema to track `execution_context` and `trigger_method`
- Natural language "analyze opportunity [Name]" still works as best-effort fallback

**Solution Implemented:**
`.claude/commands/analyze-opportunity.md` created as thin wrapper that delegates to opportunity-advancer agent spec. Zero logic duplication, deterministic trigger, discoverable via `/help`.

**Related Issue:**
#18 - Implement slash command for deterministic agent trigger

**Patterns:**
First discovery of MCP isolation in subagents. Established pattern: slash commands for MCP-dependent agents, thin wrapper pattern for zero duplication.

---

### 2025-11-29 opportunity-advancer - Vague recommendations violating Principle 5

**What Happened:**
Agent produced vague, non-actionable recommendations:
- "Follow up" without specific action or date
- Stage advancement without business justification
- No concrete next steps for user to execute

**Root Cause:**
**Principle Violations:**
1. **Principle 5 (Be Explicit, Never Vague)** - Recommendations lacked specificity
2. **Principle 8 (Strategic Context Logging)** - No memory of analysis patterns
3. **Principle 14 (User Control First)** - No validation checkpoint in REFLECT

**Technical Gap:** No structured memory to track recommendation quality or improve over time.

**Impact:**
- Severity: Medium
- Data corrupted: No (recommendations were guidance only)
- User intervention required: Yes (user had to manually refine recommendations)
- Time to recover: 5-10 minutes per opportunity

**Prevention:**
- Added `.claude/memory/opportunity-analysis-log.jsonl` for structured memory
- Enhanced PLAN phase with explicit recommendation patterns (good vs bad examples)
- Added SENSE phase MCP tool availability validation
- Added REFLECT phase Principle 14 checkpoint (user control validation)
- Added ACT phase memory logging after successful updates
- Updated PRINCIPLES.md with enforcement notes for Principles 4, 5, 8, 14

**Related Issue:**
#16 - Implement structured memory and principle enforcement

**Patterns:**
First occurrence - establishing baseline for recommendation quality tracking via memory log.

---

(Additional entries added here, newest first)

---

### Example Entry (for reference only)

### 2025-11-27 opportunity-advancer - Skipped Biz Funnel stages

**What Happened:**
Agent recommended advancing opportunity from "2. Suspect" directly to "5. Credibilidade", skipping "3. Prospect" and "4. Contato".

**Root Cause:**
Meeting transcript mentioned "building credibility" which triggered advancement to Credibilidade stage without validating sequential progression.

**Impact:**
- Severity: Medium
- Data corrupted: No (user rejected the update)
- User intervention required: Yes (manual correction)
- Time to recover: 2 minutes

**Prevention:**
Add explicit validation in PLAN phase:
- Check if advancement skips stages
- Warn user if jump detected
- Require explicit confirmation for non-sequential moves

**Related Issue:**
None (first occurrence)

**Patterns:**
None yet - monitoring for recurrence.

---
