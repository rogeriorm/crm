# Validation Results

**Date:** YYYY-MM-DD
**Phase:** 0 - Opportunity Advancer Agent Mirror
**Goal:** ≥80% match, <30sec execution, 100% business rule compliance

---

## Summary

| Metric | Result | Target | Status |
|--------|--------|--------|--------|
| Tests Passed | X/5 | ≥4/5 | ⬜ PENDING |
| Execution Time (avg) | X sec | <30 sec | ⬜ PENDING |
| Business Rules | X% | 100% | ⬜ PENDING |
| Overall Decision | - | PASS/FAIL | ⬜ PENDING |

---

## Test Results

### Test 1: #TEST Standard Update - Acme Corp
**Date:** YYYY-MM-DD
**Execution Time:** X sec
**Status:** ⬜ PENDING / ✅ PASS / ❌ FAIL

| Field | Expected | Actual | Match |
|-------|----------|--------|-------|
| Update Log | `21/11: cliente validou fit técnico` | | ⬜ |
| Next Action | `Aguardar confirmação de budget Q1` | | ⬜ |
| NAction Due | `2025-11-28` | | ⬜ |
| Biz Funnel | `5. Credibilidade` (no change) | | ⬜ |
| Status | `Waiting Feedback` | | ⬜ |
| AI Recommendation | Empty/minimal | | ⬜ |

**Business Rules:**
- [ ] Update Log ≤10 words
- [ ] APPEND to log (not replace)
- [ ] No stages skipped

**Issues:** [List any issues]

---

### Test 2: #TEST Stage Advancement - Beta Inc
**Date:** YYYY-MM-DD
**Execution Time:** X sec
**Status:** ⬜ PENDING / ✅ PASS / ❌ FAIL

| Field | Expected | Actual | Match |
|-------|----------|--------|-------|
| Update Log | `21/11: cliente solicitou proposta` | | ⬜ |
| Next Action | `Elaborar proposta comercial...` | | ⬜ |
| NAction Due | `2025-11-28` | | ⬜ |
| Biz Funnel | `6. Oferta` (**changed**) | | ⬜ |
| Reasoning | Clear justification | | ⬜ |
| Status | `In progress` | | ⬜ |
| AI Recommendation | Can be empty | | ⬜ |

**Business Rules:**
- [ ] Stage advanced correctly (5 → 6)
- [ ] No stages skipped
- [ ] Reasoning provided

**Issues:** [List any issues]

---

### Test 3: #TEST Waiting Client - Gamma Ltd
**Date:** YYYY-MM-DD
**Execution Time:** X sec
**Status:** ⬜ PENDING / ✅ PASS / ❌ FAIL

| Field | Expected | Actual | Match |
|-------|----------|--------|-------|
| Update Log | `21/11: proposta enviada, aguardando` | | ⬜ |
| Next Action | `Follow-up sobre proposta...` | | ⬜ |
| NAction Due | `2025-11-28` | | ⬜ |
| Biz Funnel | `7. Proposta` (no change) | | ⬜ |
| Status | `Waiting Feedback` (**key**) | | ⬜ |
| AI Recommendation | Can be empty | | ⬜ |

**Business Rules:**
- [ ] Status = "Waiting Feedback" (not "In progress")
- [ ] Context logged appropriately

**Issues:** [List any issues]

---

### Test 4: #TEST Multiple Interactions - Delta Co
**Date:** YYYY-MM-DD
**Execution Time:** X sec
**Status:** ⬜ PENDING / ✅ PASS / ❌ FAIL

| Field | Expected | Actual | Match |
|-------|----------|--------|-------|
| Update Log | `21/11: budget aprovado Q1...` | | ⬜ |
| Next Action | `Enviar proposta formal...` | | ⬜ |
| NAction Due | `2025-11-28` | | ⬜ |
| Biz Funnel | `7. Proposta` (**changed**) | | ⬜ |
| Reasoning | Advancement justified | | ⬜ |
| Status | `In progress` | | ⬜ |
| AI Recommendation | **Pattern: budget 3x** | | ⬜ |

**Business Rules:**
- [ ] Multi-interaction analysis performed
- [ ] Pattern detected (budget mentioned 3x)
- [ ] AI Recommendation specific and actionable

**Issues:** [List any issues]

---

### Test 5: #TEST Inline Content - Epsilon Inc
**Date:** YYYY-MM-DD
**Execution Time:** X sec
**Status:** ⬜ PENDING / ✅ PASS / ❌ FAIL

| Field | Expected | Actual | Match |
|-------|----------|--------|-------|
| Update Log | `21/11: primeira reunião...` | | ⬜ |
| Next Action | `Enviar cases de automação...` | | ⬜ |
| NAction Due | `2025-11-28` | | ⬜ |
| Biz Funnel | `5. Credibilidade` (**changed**) | | ⬜ |
| Reasoning | Building credibility phase | | ⬜ |
| Status | `In progress` | | ⬜ |
| AI Recommendation | Can be empty | | ⬜ |

**Business Rules:**
- [ ] Extracted from inline content (not Anotações relation)
- [ ] Stage advanced correctly (4 → 5)

**Issues:** [List any issues]

---

## Issues Found

### Critical Issues
[Issues that block validation success]

### Minor Issues
[Issues that don't block success but should be noted]

---

## Failure Analysis (If Applicable)

### Failed Test: [Test Name]

**Issue Description:**
[What went wrong]

**Root Cause:**
[Why did it fail? Wrong field name, logic error, data issue?]

**Proposed Fix:**
[Specific changes needed]

**Files to Change:**
- [ ] `.claude/skills/crm-data-model/SKILL.md`
- [ ] `.claude/agents/opportunity-advancer.md`
- [ ] `docs/2.2-opportunity-advancer.md`
- [ ] Other: [specify]

**Proposed Changes:**
```
[Specific changes to make]
```

**Risk:** Low / Medium / High

**Estimated Time:** X minutes

---

## Decision

**Overall Status:** ⬜ PENDING / ✅ PASS / ❌ FAIL

**Pass Criteria Met:**
- [ ] ≥4/5 tests passed
- [ ] 100% business rule compliance
- [ ] <30 sec execution per test
- [ ] No critical bugs

**Next Steps:**
- If PASS: Proceed to Phase 1 (Weekly Pipeline Digest)
- If FAIL: Implement approved fixes and re-test

---

## Validation Sign-Off

**Validated By:** [Name]
**Date:** YYYY-MM-DD
**Approved to Proceed:** YES / NO / NEEDS FIXES
