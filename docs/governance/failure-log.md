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

(Entries added here, newest first)

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
