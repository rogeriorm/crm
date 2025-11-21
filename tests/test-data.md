# Test Data Specifications

Create these 5 test opportunities in Notion Oportunidades database.

**IMPORTANT:** All Nome Oportunidade fields MUST start with `#TEST` prefix.

---

## Test 1: Standard Update (Happy Path)

**Nome Oportunidade:** `#TEST Standard Update - Acme Corp`

**Properties:**
- Biz Funnel: `5. Credibilidade`
- Status: `In progress`
- Priority: `Medium`
- Update Log: (leave empty)
- Next Action: (leave empty)
- NAction Due: (leave empty)

**Create Anotações (linked page):**
- Title: `@21/11 10:00 AM (GMT-3)`
- Date property: `2025-11-21`
- Content:
```
<summary>
Cliente validou fit técnico da solução proposta. Demonstrou interesse em implementação rápida para Q1 2025. Próximos passos: aguardar aprovação interna do budget Q1. Timeline esperado: 1 semana para resposta.
</summary>
```

---

## Test 2: Stage Advancement Detection

**Nome Oportunidade:** `#TEST Stage Advancement - Beta Inc`

**Properties:**
- Biz Funnel: `5. Credibilidade`
- Status: `In progress`
- Priority: `High`

**Create Anotações:**
- Title: `@21/11 14:30 PM (GMT-3)`
- Date: `2025-11-21`
- Content:
```
<summary>
Cliente perguntou "quanto custaria uma implementação completa para nossa operação?" e solicitou envio de proposta de investimento. Mencionou que tem budget aprovado para Q1 2025. Quer proposta até final do mês.
</summary>
```

---

## Test 3: Waiting on Client

**Nome Oportunidade:** `#TEST Waiting Client - Gamma Ltd`

**Properties:**
- Biz Funnel: `7. Proposta`
- Status: `In progress`
- Priority: `Medium`

**Create Anotações:**
- Title: `@21/11 16:00 PM (GMT-3)`
- Date: `2025-11-21`
- Content:
```
<summary>
Proposta comercial enviada por email. Cliente confirmou recebimento e disse que vai revisar internamente com o time técnico e financeiro. Previsão de resposta: próxima semana. Aguardando feedback do cliente.
</summary>
```

---

## Test 4: Multiple Interactions (Pattern Detection)

**Nome Oportunidade:** `#TEST Multiple Interactions - Delta Co`

**Properties:**
- Biz Funnel: `6. Oferta`
- Status: `In progress`
- Priority: `High`

**Create 3 Anotações (oldest to newest):**

**Anotação 1:**
- Title: `@14/11 10:00 AM (GMT-3)`
- Date: `2025-11-14`
- Content:
```
<summary>
Cliente mencionou preocupação com budget limitado. Precisa validar ROI antes de aprovar investimento. Discussão focada em custos e benefícios.
</summary>
```

**Anotação 2:**
- Title: `@18/11 14:00 PM (GMT-3)`
- Date: `2025-11-18`
- Content:
```
<summary>
Cliente perguntou sobre formas de pagamento e parcelamento. Mencionou novamente a questão do budget. Parece estar buscando aprovação interna.
</summary>
```

**Anotação 3:**
- Title: `@21/11 11:00 AM (GMT-3)`
- Date: `2025-11-21`
- Content:
```
<summary>
Cliente confirmou que budget foi aprovado para Q1 2025! Esta é a terceira vez que budget foi mencionado nas conversas. Agora quer proposta formal detalhada com todas opções discutidas.
</summary>
```

---

## Test 5: Inline Content Detection

**Nome Oportunidade:** `#TEST Inline Content - Epsilon Inc`

**Properties:**
- Biz Funnel: `4. Contato`
- Status: `In progress`
- Priority: `Low`
- **Anotações relation:** (leave EMPTY - no linked pages)

**Add inline content directly to opportunity page:**
```
<summary>
Primeira reunião de alinhamento realizada. Cliente demonstrou interesse em automação de processos internos. Identificamos oportunidade de melhorar eficiência operacional. Próximo passo: enviar material sobre cases similares do setor para construir credibilidade.
</summary>
```

---

## Setup Checklist

- [ ] Test 1 created with 1 linked Anotação
- [ ] Test 2 created with 1 linked Anotação (advancement signal)
- [ ] Test 3 created with 1 linked Anotação (waiting scenario)
- [ ] Test 4 created with 3 linked Anotações (pattern detection)
- [ ] Test 5 created with inline content (no Anotações relation)
- [ ] All Nome Oportunidade start with `#TEST` prefix
- [ ] All dates set to 2025-11-21 (or as specified)

---

## Quick Verification

Search in Notion: `#TEST`
Should return exactly 5 opportunities.
