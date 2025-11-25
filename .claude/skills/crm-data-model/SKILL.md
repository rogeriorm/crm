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

## Meeting Data Patterns

**Inline:** Content has `<transcript>`, `<summary>`, `<notes>` tags
**Linked:** Anotações property contains page URLs
**Priority:** Check inline first, then linked relations
**Content hierarchy:** transcript > summary > notes

## Update Log Format

`DD/MM: [key decision/action/result]` - Max 10 words, newest first

**Log only:** Decisions, advancement signals, stage triggers, blockers
**Never log:** Generic meetings, vague descriptions, no-value entries
**CRITICAL:** Always APPEND, never replace

## Strategic Analysis

### Advancement Signals (Biz Funnel Stage Transitions)
- **Credibilidade → Oferta:** "how much?" or pricing questions
- **Oferta → Proposta:** formal proposal request
- **Proposta → Negociação:** discussing terms/adjustments
- **Negociação → Fechamento:** purchase confirmation/signature
- **RULE:** Never skip stages

### Pattern Detection Priority
1. Recent interactions WITH advancement signals
2. Multi-interaction patterns (escalating interest, stalling, triggers)
3. Strategic context (blockers, dependencies, competition)

### AI Recommendation Format
- 2-3 sentences max, data-driven, actionable
- Focus: patterns across interactions, opportunities, risks, blind spots
- Can be empty if no significant insights

### Recommendation Patterns
- Pattern recognition: "X mentioned 3x + Y confirmed = ideal for Z"
- Opportunity: "High response rate + defined timeline = accelerate"
- Risk: "No movement 21d despite interest = proactive unblock needed"
- Strategic: "Timeline requires proposal by [date] to enable implementation"
- Avoid: Generic statements without specific data or actions

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

---

## Opportunity Search Protocol

**Purpose:** Ensure agents use validated database ID and handle multiple results correctly.

### Search Rules

1. **Always use validated database ID:**
   - Correct: `collection://201b1882-308d-4524-8a86-6672d5502299`
   - NEVER use: shortcuts, aliases, or unvalidated IDs

2. **Multiple results handling:**
   - If search returns multiple matches → show ALL to user for selection
   - NEVER auto-select first result
   - Display: Name, Biz Funnel stage, Status for each
   - Wait for user to choose by number

3. **Search validation:**
   - Before searching: verify database ID is correct
   - After searching: log result count
   - If 0 results: clear error message with guidance
   - If 1 result: proceed (but confirm with user)
   - If 2+ results: trigger selection flow

### Example Flow

```
SENSE Phase:
1. User provides: "opportunity ABC"
2. Agent validates: database ID = collection://201b1882-308d-4524-8a86-6672d5502299
3. Agent searches with notion-search
4. Results: 3 opportunities found
5. Agent displays:
   "Found 3 opportunities matching 'ABC':
   1. ABC Corp - Oferta - In Progress
   2. ABC Industries - Proposta - Waiting Feedback
   3. ABC Holdings - Credibilidade - Scheduled

   Which opportunity? (enter number)"
6. User selects: 2
7. Agent proceeds with ABC Industries
```

### Memory Logging

After search, log to `.claude/memory/opportunity-search-log.jsonl`:
```json
{
  "timestamp": "ISO-8601",
  "search_term": "user input",
  "database_id": "collection://...",
  "database_validated": true,
  "result_count": 3,
  "selected_index": 2,
  "selected_name": "ABC Industries"
}
```
