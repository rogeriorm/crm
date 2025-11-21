# Test Cases - Expected Outputs

Run opportunity-advancer agent on each test opportunity and compare actual vs expected.

---

## Test 1: #TEST Standard Update - Acme Corp

**Validates:** Basic SPAR cycle, 6-field generation

**Expected Output:**

| Field | Expected Value | Validation |
|-------|---------------|------------|
| **Update Log** | `21/11: cliente validou fit técnico` | ≤10 words, strategic |
| **Next Action** | `Aguardar confirmação de budget Q1` | Specific, actionable |
| **NAction Due** | `2025-11-28` | Meeting date + 7 days |
| **Biz Funnel** | `5. Credibilidade` | No change (no advancement signal) |
| **Status** | `Waiting Feedback` | Waiting on client budget approval |
| **AI Recommendation** | Empty or minimal | Only 1 interaction, no patterns yet |

**Business Rules:**
- [ ] Update Log ≤10 words
- [ ] Next Action actionable
- [ ] NAction Due = 2025-11-21 + 7 days
- [ ] Biz Funnel unchanged
- [ ] APPEND to Update Log (not replace)

---

## Test 2: #TEST Stage Advancement - Beta Inc

**Validates:** Biz Funnel advancement detection, reasoning

**Expected Output:**

| Field | Expected Value | Validation |
|-------|---------------|------------|
| **Update Log** | `21/11: cliente solicitou proposta de investimento` | Key signal logged |
| **Next Action** | `Elaborar proposta comercial com pricing Q1` | Specific, actionable |
| **NAction Due** | `2025-11-28` | Meeting + 7 days |
| **Biz Funnel** | `6. Oferta` | **CHANGED** from 5. Credibilidade |
| **Reasoning** | `Cliente perguntou "quanto custa" - sinal de avanço Credibilidade → Oferta` | Clear justification |
| **Status** | `In progress` | User has action (create proposal) |
| **AI Recommendation** | Can be empty | Only 1 interaction |

**Business Rules:**
- [ ] Biz Funnel advanced correctly (5 → 6)
- [ ] No stages skipped
- [ ] Reasoning provided for stage change
- [ ] Update Log reflects advancement signal

---

## Test 3: #TEST Waiting Client - Gamma Ltd

**Validates:** Status decision logic (Waiting Feedback)

**Expected Output:**

| Field | Expected Value | Validation |
|-------|---------------|------------|
| **Update Log** | `21/11: proposta enviada, aguardando análise` | Context logged |
| **Next Action** | `Follow-up sobre proposta após análise técnica` | Describes next step |
| **NAction Due** | `2025-11-28` | Meeting + 7 days |
| **Biz Funnel** | `7. Proposta` | No change |
| **Status** | `Waiting Feedback` | **KEY:** Waiting on client, not user |
| **AI Recommendation** | Can be empty | Only 1 interaction |

**Business Rules:**
- [ ] Status = "Waiting Feedback" (not "In progress")
- [ ] Next Action describes what to do AFTER client responds
- [ ] Update Log explains waiting context

---

## Test 4: #TEST Multiple Interactions - Delta Co

**Validates:** Multi-source analysis, pattern recognition, AI recommendation

**Expected Output:**

| Field | Expected Value | Validation |
|-------|---------------|------------|
| **Update Log** | `21/11: budget aprovado Q1, pronto para proposta` | Strategic context from multiple interactions |
| **Next Action** | `Enviar proposta formal com opções de pagamento` | Specific, incorporates pattern (payment options) |
| **NAction Due** | `2025-11-28` | Most recent meeting + 7 days |
| **Biz Funnel** | `7. Proposta` | **CHANGED** from 6. Oferta |
| **Reasoning** | `Cliente confirmou budget e solicitou proposta formal` | Advancement justified |
| **Status** | `In progress` | User has action |
| **AI Recommendation** | `Cliente mencionou budget em 3 interações recentes e agora confirmou aprovação - momento ideal para proposta formal. Incluir opções de pagamento na proposta.` | **KEY:** Pattern detected across 3 meetings |

**Business Rules:**
- [ ] Agent analyzed multiple interactions (not just latest)
- [ ] AI Recommendation identifies pattern (budget mentioned 3x)
- [ ] AI Recommendation is specific and actionable
- [ ] Biz Funnel advanced correctly (6 → 7)

---

## Test 5: #TEST Inline Content - Epsilon Inc

**Validates:** Detection priority (inline content before linked Anotações)

**Expected Output:**

| Field | Expected Value | Validation |
|-------|---------------|------------|
| **Update Log** | `21/11: primeira reunião, interesse em automação` | Extracted from inline content |
| **Next Action** | `Enviar cases de automação de processos similares` | From inline summary |
| **NAction Due** | `2025-11-28` | Estimated meeting date + 7 days |
| **Biz Funnel** | `5. Credibilidade` | **CHANGED** from 4. Contato |
| **Reasoning** | `Iniciando construção de credibilidade com cases` | Building trust phase |
| **Status** | `In progress` | User has action |
| **AI Recommendation** | Can be empty | Only 1 interaction |

**Business Rules:**
- [ ] Agent extracted content from inline (not Anotações relation)
- [ ] Biz Funnel advanced correctly (4 → 5)
- [ ] Update Log extracted from inline summary

---

## Overall Success Criteria

**PASS if:**
- ≥4/5 tests succeed
- 100% business rule compliance
- All execution times <30 seconds
- No critical bugs

**FAIL if:**
- <4/5 tests pass
- Any business rule violations
- Critical bugs found

---

## Validation Checklist (Per Test)

For each test case, check:
- [ ] 6 fields generated
- [ ] Update Log ≤10 words
- [ ] Next Action specific and actionable
- [ ] NAction Due = meeting date + 7 days
- [ ] Biz Funnel correct (with/without advancement)
- [ ] Status makes sense (who has action?)
- [ ] AI Recommendation appropriate (or empty if 1 interaction)
- [ ] Execution time <30 seconds
- [ ] Business rules followed (APPEND, no skip stages, etc.)
