# CRM Data Model

**Activate when:** User mentions opportunities, pipeline, Biz Funnel, CRM workflow, or Notion databases

## Oportunidades Database

**Collection ID:** `collection://201b1882-308d-4524-8a86-6672d5502299`

**🔗 Direct Access:** [Open in Notion](https://www.notion.so/201b1882308d45248a866672d5502299)

**Purpose:** Track business opportunities through sales funnel

## Fields Reference

| Field | Type | Notes |
|-------|------|-------|
| **Nome Oportunidade** | title | Primary identifier |
| **Biz Funnel** | select | 11 stages (see below) |
| **Status** | status | 8 options (see below) |
| **Priority** | select | High / Medium / Low |
| **BTAG** | multi-select | Business tags |
| **Update Log** | text | Newest first, DD/MM format, APPEND only |
| **Next Action** | text | Verb + object, specific |
| **NAction Due** | date | Use `date:NAction Due:start` format |
| **AI Advancement Recommendation** | text | 2-3 sentences, can be empty |
| **Due** | date | Overall deadline |
| **Days in Stage** | formula | Read-only, auto-calculated |
| **Days Until Due** | formula | Read-only, auto-calculated |
| **Anotações** | relation | Meeting notes (inline or linked) |
| **relTasksNotes** | relation | Secondary notes link |
| **Contatos** | relation | People |
| **Cliente** | relation | Companies |
| **Source** | relation | Opportunity source |
| **Last Edited Time** | system | Auto-tracked |

### Field Usage Notes

**Update Log:** Always APPEND new entries, never replace. Format: `DD/MM: [max 10 words]`

**NAction Due:** Use MCP format: `"date:NAction Due:start": "YYYY-MM-DD"`, `"date:NAction Due:is_datetime": 0`

**AI Advancement Recommendation:** Strategic insights from multi-interaction analysis. Optional (can be empty).

**Formulas (Days in Stage, Days Until Due):** Read-only, do not attempt to update.

**Anotações:** Can be inline (page content) or linked (relation URLs). Check both patterns.

## Biz Funnel Stages (11 stages)

**Flow:** Marketing → Suspect → Prospect → Contato → Credibilidade → Oferta → Proposta → Negociação → Fechamento → Relacionamento → done!

**Most active stages:** Credibilidade, Oferta, Proposta, Negociação

### Stage Definitions
1. **Marketing:** General awareness, no specific lead yet
2. **Suspect:** Potential lead identified, not qualified
3. **Prospect:** Qualified lead, fit confirmed
4. **Contato:** First contact made, relationship starting
5. **Credibilidade:** Building trust, demonstrating expertise
6. **Oferta:** Client interested, discussing solutions/pricing
7. **Proposta:** Formal proposal presented
8. **Negociação:** Negotiating terms, price, timeline
9. **Fechamento:** Closing the deal, final signatures
10. **Relacionamento:** Ongoing relationship, delivery phase
11. **done!:** Project completed

### Advancement Signals
- **Credibilidade → Oferta:** Client asks "how much?" or "what would this cost?"
- **Oferta → Proposta:** Client requests formal proposal document
- **Proposta → Negociação:** Client says "can we adjust X?" or discusses terms
- **Negociação → Fechamento:** Client confirms purchase or signs contract

**RULE:** Never skip stages (e.g., cannot go Credibilidade → Proposta directly)

## Status Options

**8 options in 3 groups:**
- **To-do:** Not started, In Planning
- **In Progress:** Scheduled, On Hold, Waiting Feedback, In progress
- **Complete:** Done, Archived

### Primary Status Decision Logic
- User has action to take → **"In progress"**
- Awaiting client response → **"Waiting Feedback"**
- Next meeting booked → **"Scheduled"**
- Temporarily paused → **"On Hold"**

## Meeting Data Storage Patterns

Customer interactions stored in 2 patterns:

### Pattern 1: Inline Anotações (Embedded in Page Content)
- Meeting content embedded directly in Opportunity page content
- Structure: Content includes `<transcript>`, `<summary>`, `<notes>` XML tags
- Use case: Quick access, single-page view, lightweight interactions
- MCP fetch behavior: Page content includes meeting data inline

### Pattern 2: Linked Anotações (Separate Pages)
- Anotações property contains relation URLs to separate pages
- Each URL points to a page in Anotações database
- Structure: Each page has `<transcript>`, `<summary>`, `<notes>` tags
- Use case: Multiple meetings, better organization, auto-generated via Notion AI
- MCP fetch behavior: Anotações property returns array of page URLs

**Anotações Page Properties:**
- **Date** (date): When meeting occurred
- **Title** (title): Often format "@Today 11:11 AM (GMT-3)" or "@DD/MM HH:MM AM/PM"

### Detection Priority (for agents)
When loading meeting data:
1. **First:** Check Opportunity page content for inline `<transcript>`, `<summary>`, or `<notes>` tags
2. **Second:** Check Anotações relation property for linked pages

### Content Tags
- `<transcript>`: Full meeting transcript (auto-generated or manual)
- `<summary>`: AI-generated summary of key points
- `<notes>`: User notes and observations

### Content Loading Priority
When extracting content from a meeting interaction:
1. Use `<transcript>` if available (most detailed)
2. Fallback to `<summary>` if no transcript
3. Fallback to `<notes>` if no summary

## Update Log Format & Strategic Principles

**Format:** Reverse chronological (newest first)

**Entry Format:** `DD/MM: [action or result]`

**Core Rules:**
- Max 10 words per entry
- Focus on KEY decision/action/result only
- NOT a meeting summary

**Strategic Principles:**

### For Interactions WITH Advancement
Log the KEY signal that indicates progress:
- ✅ "18/11: cliente aprovou proposta técnica" (DECISION)
- ✅ "15/11: cliente confirmou budget aprovado" (ADVANCEMENT SIGNAL)
- ✅ "12/11: cliente solicitou proposta formal" (STAGE TRIGGER)

### For Interactions WITHOUT Advancement
Log the strategic CONTEXT/REASON:
- ✅ "18/11: cliente aguardando aprovação do board" (EXPLAINS DELAY)
- ✅ "15/11: definido timeline para decisão até 30/11" (CONTEXTUAL INFO)
- ✅ "12/11: validado fit técnico, aguarda budget Q1" (PROGRESS + BLOCKER)

### What NOT to Log
- ❌ "18/11: tivemos reunião muito boa" (NO INFORMATION)
- ❌ "15/11: call de acompanhamento realizado" (NO VALUE)
- ❌ "12/11: discutimos diversos aspectos do projeto" (VAGUE)

**CRITICAL RULE:** Always APPEND new entries, never replace existing log

## Interaction Analysis Strategy

**Goal:** Extract strategic insights from customer interactions, not just summarize meetings

### Multi-Source Pattern Recognition

When analyzing opportunities:
1. **Load multiple recent interactions** (2-3 most recent, not just one)
2. **Analyze across all sources** for patterns and signals
3. **Prioritize strategic content** over chronological recency
4. **Identify advancement patterns** across multiple touchpoints

### Pattern Categories to Detect

**Escalating Interest:**
- Questions becoming more specific/detailed
- Client proactively following up
- Mentions of timeline, budget, implementation

**Stalling Signals:**
- No concrete commitments across multiple interactions
- Repeated postponements
- Generic responses without advancement

**Advancement Triggers:**
- Repeated mentions of same topic (e.g., pricing mentioned 3x)
- Client asking "how" or "when" questions
- Introduction to decision-makers
- Budget/timeline confirmations

**Strategic Context:**
- External blockers (board approval, budget cycles)
- Internal dependencies (technical validation, legal review)
- Competitive dynamics

### Interaction Selection Logic

**Default:** Use most recent interaction by date/timestamp

**Strategic override:**
- If most recent is just "check-in" with no signals → look back for meaningful content
- Prioritize interactions with decisions, commitments, or stage signals
- Consider synthesizing insights from 2-3 recent interactions

**Hierarchy:**
1. Most recent interaction WITH advancement signals
2. If none, analyze most recent for implicit signals or strategic context
3. If truly no signals, focus Next Action on creating advancement opportunity

## AI Advancement Recommendation Guidelines

**Purpose:** Provide strategic insights the user might miss by analyzing patterns across interactions

### Content Categories

**1. Pattern Recognition**
- "Cliente mencionou orçamento em 3 interações recentes - sinal forte de interesse em proposta formal"
- "Perguntas evoluindo de 'quanto custa' para 'como implementar' - indica avanço de Oferta → Proposta"

**2. Opportunity Detection**
- "Momento ideal para proposta detalhada: budget confirmado + timeline definido"
- "Cliente respondeu rapidamente em todas interações - alta prioridade, acelerar processo"

**3. Risk Warnings**
- "Sem avanço há 14 dias apesar de cliente responsivo - considerar proposta mais proativa"
- "Sinais mistos: interesse alto mas sem compromissos concretos - validar budget e autoridade"
- "Cliente adiou reunião 2x sem reagendar - possível perda de prioridade"

**4. Blind Spots / Strategic Suggestions**
- "Considere validar autoridade de decisão antes de investir em proposta detalhada"
- "Cliente mencionou concorrente X - preparar diferenciação para próxima interação"
- "Timeline do cliente (Q1) requer proposta até [data] para viabilizar implementação"

### Format Rules

- **Length:** 2-3 sentences maximum
- **Specificity:** Based on data patterns from interactions, not speculation
- **Actionability:** Should inform strategy or next actions
- **Optional:** Can be left empty if no significant insights to add
- **Tone:** Direct and consultative, not salesy

### Examples

**Good:**
```
"Cliente mencionou orçamento 3x e agora confirmou aprovação - momento ideal para
proposta formal detalhada. Considere incluir case study similar para reforçar
credibilidade e acelerar decisão."
```

**Good:**
```
"Sem movimento há 21 dias mas cliente mantém interesse (respondeu última mensagem
rapidamente). Sugerir reunião objetiva para validar timeline e desbloquear processo."
```

**Good:**
```
"Padrão detectado: cliente está avaliando 2 fornecedores e pediu proposta de ambos.
Destacar diferenciais técnicos e referências do setor na proposta para criar vantagem."
```

**Avoid:**
```
"Cliente parece interessado. Continue acompanhando."
(Too generic, no specific insight or action)
```

## Date Property Format

When updating date properties via MCP:
```json
{
  "date:NAction Due:start": "YYYY-MM-DD",
  "date:NAction Due:is_datetime": 0
}
```

**Note:** Field name is "NAction Due" (not "Next Action Date")

## Common Queries

**Search by name:**
```
mcp__notion__notion-search
query: "opportunity name"
data_source_url: "collection://201b1882-308d-4524-8a86-6672d5502299"
```

**Fetch opportunity:**
```
mcp__notion__notion-fetch
id: {page_url or page_id}
```

**Update properties:**
```
mcp__notion__notion-update-page
page_id: {page_id}
command: "update_properties"
properties: {
  "Update Log": "{new_entry}\n{existing_log}",
  "Next Action": "...",
  "date:NAction Due:start": "YYYY-MM-DD",
  "date:NAction Due:is_datetime": 0,
  "Biz Funnel": "...",
  "Status": "...",
  "AI Advancement Recommendation": "..."
}
```
