# Test Cases for Issue #17: Data Traceability

**Issue:** Add Notion URL display to opportunity-advancer outputs
**Date:** 2025-11-29
**Principle:** 26 (Data Traceability)

---

## Test Case 1: Verify Principle 26 Documentation

**Objective:** Confirm Principle 26 is properly documented in PRINCIPLES.md with clear examples and requirements.

**Steps:**
1. Open `docs/PRINCIPLES.md`
2. Locate Principle 26 section
3. Verify it contains:
   - Clear rule statement
   - "Why this matters" rationale
   - Examples of CORRECT behavior (with ✅)
   - Examples of VIOLATIONS (with ❌)
   - Implementation requirements for SENSE and ACT phases
   - Error handling guidelines
   - Exceptions (aggregate metrics, etc.)

**Expected Result:**
- [ ] Principle 26 exists in PRINCIPLES.md (line ~679)
- [ ] Contains rule: "Every agent output that presents data MUST include direct, clickable links"
- [ ] Has 3+ examples of correct behavior (Opportunity Analysis, Pipeline Digest, Contact Summary)
- [ ] Has 2+ examples of violations (Missing Link, Name Only)
- [ ] Documents implementation requirements for SENSE and ACT phases
- [ ] Version history updated to v1.2 (2025-11-29)
- [ ] Last Updated date shows 2025-11-29

**Pass Criteria:** All checkboxes checked ✓

---

## Test Case 2: Verify opportunity-advancer Compliance

**Objective:** Confirm opportunity-advancer agent specification enforces Principle 26 in both ACT and REFLECT phases.

**Steps:**
1. Open `.claude/agents/opportunity-advancer.md`
2. Check ACT phase section (around line 174-184)
3. Check REFLECT phase section (around line 201-214)
4. Check Output Format section (around line 243-250)

**Expected Result:**

### ACT Phase Compliance:
- [ ] Step 1 explicitly mentions "CRITICAL (Principle 26: Data Traceability)"
- [ ] Instructions to extract URL from SENSE phase fetch response
- [ ] Instructions to display URL in blockquote format
- [ ] Requirement states "User MUST be able to click directly to Notion record"

### REFLECT Phase Compliance:
- [ ] New checkpoint section titled "Principle 26 Checkpoint (Data Traceability)"
- [ ] Checkpoint asks: "Did output include Notion URL for user validation?"
- [ ] Checkpoint asks: "Is URL clickable and placed prominently at top of output?"
- [ ] Checkpoint asks: "Did URL come from MCP fetch response (not manually constructed)?"

### Output Format Compliance:
- [ ] Output format template includes `> **Notion URL:** [Direct link to opportunity page]`
- [ ] URL placement is prominently at top (before analysis content)
- [ ] Format uses blockquote (>) for visual emphasis

**Pass Criteria:** All checkboxes checked ✓

---

## Test Case 3 (Manual - Future): Live Agent Execution Test

**Objective:** Verify that when opportunity-advancer runs, it actually displays the Notion URL in output.

**Prerequisites:**
- MCP Notion integration active
- Access to Oportunidades database

**Steps:**
1. Run opportunity-advancer with test opportunity name
2. Agent should execute SENSE phase and fetch opportunity data
3. Agent should display analysis output

**Expected Output Format:**
```markdown
# Opportunity Update: [Exact Name from Notion]

> **Oportunidade:** [Name]
> **Notion URL:** https://www.notion.so/[page-id]

**Last Interaction:** ...
**Current Stage:** ...

[rest of analysis]
```

**Validation:**
- [ ] URL is displayed prominently at top
- [ ] URL is clickable (markdown link format)
- [ ] URL points to correct Notion page
- [ ] URL comes from MCP fetch response (not constructed)

**Pass Criteria:** URL displays correctly in agent output

**Status:** DEFERRED - This test requires live agent execution and will be validated when user next runs opportunity-advancer

---

## Summary

**Tests to Run Now:** 2 (Test Case 1 & 2 - Documentation verification)
**Tests Deferred:** 1 (Test Case 3 - Live execution, user will validate)

**Sign-off:**
- [ ] Test Case 1: PASSED
- [ ] Test Case 2: PASSED
- [ ] Ready to commit: YES/NO

---

## Validation Notes

_Fill in after running tests:_

**Test Case 1 Results:**


**Test Case 2 Results:**


**Issues Found:**


**Ready for Commit:** YES / NO
