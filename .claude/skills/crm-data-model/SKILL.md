# CRM Data Model

**Activate when:** User mentions opportunities, pipeline, Biz Funnel, CRM workflow, or Notion databases

## Oportunidades Database

**Collection ID:** `collection://201b1882-308d-4524-8a86-6672d5502299`

**Purpose:** Track business opportunities through sales funnel

## Key Properties

### Core Fields
- **Task** (title): Opportunity name/description
- **Biz Funnel** (select): Current sales stage (see below)
- **Status** (select): Workflow state
- **Priority** (select): High | Medium | Low

### Action Fields
- **Update Log** (text): Chronological updates, newest first, format "DD/MM: [action]"
- **Next Action** (text): Specific next step (verb + object)
- **Next Action Date** (date): When to act

### Related Data
- **Anotações** (relation): Meeting notes with transcripts/summaries
- **Offline Notes** (text): Manual notes (fallback if no Anotações)
- **Contatos** (relation): People involved
- **Cliente** (relation): Company/organization

### Calculated Fields
- **Days in Stage** (formula): Time in current Biz Funnel stage
- **Days Until Due** (formula): Time remaining to deadline
- **Due** (date): Deadline/target close date

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

- **In Progress:** User has action to take right now
- **Waiting Feedback:** Awaiting client response (no action for user)
- **Scheduled:** Next action/meeting has confirmed date/time
- **On Hold:** Temporarily paused (budget freeze, timing issues)
- **Lost:** Deal did not close
- **Won:** Deal closed successfully

### Status Decision Logic
- If Next Action is something user must do → "In Progress"
- If waiting on client decision/response → "Waiting Feedback"
- If next meeting is booked → "Scheduled"

## Anotações (Meeting Notes) Structure

**Content Tags:**
- `<transcript>`: Full meeting transcript (auto-generated or manual)
- `<summary>`: AI-generated summary of key points
- `<notes>`: User notes and observations

**Properties:**
- **Date** (date): When meeting occurred
- **Title** (title): Often format "@Today 11:11 AM (GMT-3)" or "@DD/MM HH:MM AM/PM"

**Loading Priority:**
1. Use `<transcript>` if available (most detailed)
2. Fallback to `<summary>` if no transcript
3. Fallback to `<notes>` if no summary
4. Last resort: Opportunity's "Offline Notes" field

## Update Log Format

**Format:** Reverse chronological (newest first)

**Entry Format:** `DD/MM: [action or result]`

**Rules:**
- Max 10 words per entry
- Focus on KEY decision/action/result only
- NOT a full summary
- Examples:
  - ✅ "18/11: cliente aprovou proposta técnica"
  - ✅ "15/11: enviada proposta comercial revisada"
  - ❌ "18/11: tivemos reunião muito boa onde discutimos diversos aspectos e cliente gostou"

**IMPORTANT:** Always APPEND new entries, never replace existing log

## Date Property Format

When updating date properties via MCP:
```json
{
  "date:Next Action Date:start": "YYYY-MM-DD",
  "date:Next Action Date:is_datetime": 0
}
```

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
  "date:Next Action Date:start": "YYYY-MM-DD",
  "Biz Funnel": "...",
  "Status": "..."
}
```
