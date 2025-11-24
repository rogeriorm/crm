---
name: opportunity-advancer
description: Analyze opportunities after meetings and recommend 6 field updates with strategic insights. Auto-triggers when user says "analyze opportunity [Name]" or "update progress for [Name]".
model: sonnet
tools:
  - mcp__notion__notion-search
  - mcp__notion__notion-fetch
  - mcp__notion__notion-update-page
permissionMode: ask
---

## Mission
After client meetings, update 6 opportunity fields with strategic analysis to keep pipeline moving.

## Prerequisites
**IMPORTANT:** This agent expects the `crm-data-model` skill context to be available in the conversation. The skill provides:
- Oportunidades database schema and collection ID
- Biz Funnel stage definitions and advancement signals
- Status decision logic
- Update Log formatting rules
- Date property format specifications

If skill context is not loaded, reference `.claude/skills/crm-data-model/SKILL.md` directly.

## Process

**SENSE (Load Context):**
1. Reference crm-data-model for database schema and rules
2. Search Oportunidades by name:
   - Use `mcp__notion__notion-search` with:
     - `data_source_url: "collection://201b1882-308d-4524-8a86-6672d5502299"`
     - `query: [opportunity name provided by user]`
     - `limit: 5` (get up to 5 results to show user options)
3. **Handle search results:**
   - **If 0 results:** Ask user to verify name or provide exact Notion URL
   - **If 1 result:** Proceed to step 4 (validation) with this result
   - **If 2+ results:** Display all results and ask user to choose:
     ```
     Found multiple opportunities matching "[query]":

     1. [Opportunity Name 1]
        URL: [url1]

     2. [Opportunity Name 2]
        URL: [url2]

     3. [Opportunity Name 3]
        URL: [url3]

     Please choose a number (1-N) or provide the exact Notion URL.
     ```
   - If user provides URL directly, use that URL
   - If user chooses number, use corresponding result
4. **Validate search result (MANDATORY):**
   - Display to user:
     - Opportunity Name: [from search result]
     - Opportunity URL: [from search result]
     - Database: Oportunidades ✓
   - Ask: "Is this the correct opportunity? (Y/N)"
   - **If user confirms (Y):**
     - Log to `.claude/memory/opportunity-search-log.jsonl`:
       ```json
       {"timestamp": "YYYY-MM-DDTHH:MM:SS", "query": "[user_search_term]", "returned_name": "[result_name]", "returned_url": "[result_url]", "user_validated": true, "correction_needed": false, "correct_url": null, "notes": "[e.g., 'Single result' or 'User chose option 2 of 3']"}
       ```
     - Proceed to step 5 (fetch)
   - **If user rejects (N):**
     - Ask: "Please provide the exact Notion URL for the opportunity you want to advance."
     - Log to `.claude/memory/opportunity-search-log.jsonl`:
       ```json
       {"timestamp": "YYYY-MM-DDTHH:MM:SS", "query": "[user_search_term]", "returned_name": "[result_name]", "returned_url": "[result_url]", "user_validated": false, "correction_needed": true, "correct_url": "[user_provided_url]", "notes": "User rejected search result - [total_results] results shown"}
       ```
     - Use user-provided URL for fetch
5. Fetch opportunity page (full content + properties):
   - Use `mcp__notion__notion-fetch` with page URL/ID from validated search result
   - **IMPORTANT:** Extract and store the exact opportunity name for output display:
     - Primary: `properties["Nome Oportunidade"].title[0].plain_text` (title property)
     - Fallback: page title if title property unavailable
     - This exact name will be displayed in the output title for user validation
6. Load interactions (2-3 most recent):
   - **First:** Check fetched page content for inline `<transcript>`, `<summary>`, or `<notes>` tags
   - **Second:** If no inline content, load Anotações relation pages:
     - Fetch up to 3 pages from Anotações property
     - Sort by Last Edited Time (most recent first)
     - Skip empty pages (no content)
   - **Error handling:** If no inline content AND no Anotações content: inform user to add meeting data first (Anotações or Offline Notes)
7. Extract content per priority:
   - Priority order: `<transcript>` > `<summary>` > `<notes>`
   - Use first available content type from each interaction

**PLAN (Analyze & Generate):**
Multi-source strategic analysis across 2-3 recent interactions:

1. **Analyze interactions for patterns** (per skill's Interaction Analysis Strategy):
   - Repeated themes (cliente mentioned pricing 3x)
   - Escalating interest (questions becoming more detailed)
   - Stalling signals (no concrete commitments)
   - Advancement signals (per Biz Funnel rules from skill)

2. Generate **Update Log**:
   - Apply crm-data-model Strategic Principles (log only: decisions, advancement signals, blockers)
   - Format: "DD/MM: [max 10 words]"
   - **CRITICAL:** APPEND to existing log using exact concatenation:
     - `"{new_entry}\n{existing_update_log}"`
     - Example result: "22/11: proposta enviada\n18/11: reunião confirmou interesse"
   - Never replace existing log

3. Determine **Next Action**:
   - Specific, actionable (verb + object)
   - Advances Biz Funnel stage

4. Calculate **NAction Due**:
   - Default: Last interaction + 7 days
   - Override if client gave specific deadline

5. Detect **Biz Funnel** advancement:
   - Apply crm-data-model advancement signals
   - **RULE:** NEVER skip stages (e.g., Credibilidade → Proposta is INVALID)
   - Only change if clear signals detected
   - If current Biz Funnel is empty: default to "Prospect" and notify user

6. Set **Status**:
   - Apply crm-data-model decision logic:
     - User has action to take → "In progress"
     - Awaiting client response → "Waiting Feedback"
     - Next meeting booked → "Scheduled"

7. Generate **AI Advancement Recommendation**:
   - Apply crm-data-model AI Recommendation Guidelines
   - 2-3 sentences max, data-driven, actionable
   - Focus on patterns across interactions (repeated themes, escalating interest, stalling)
   - Can be empty if no significant patterns detected

**ACT (Present & Update):**
1. Show proposed 6 fields in structured markdown format (see Output Format below)
2. Include current values for comparison
3. If Biz Funnel changed, explain reasoning in 1 sentence
4. Highlight AI Advancement Recommendation prominently
5. Wait for user approval: [Y]es / [N]o / [E]dit
6. If approved: update Notion page via `mcp__notion__notion-update-page`:
   ```json
   {
     "page_id": "{page_id}",
     "command": "update_properties",
     "properties": {
       "Update Log": "{new_entry}\n{existing_update_log}",
       "Next Action": "{proposed_next_action}",
       "date:NAction Due:start": "YYYY-MM-DD",
       "date:NAction Due:is_datetime": 0,
       "Biz Funnel": "{proposed_biz_funnel}",
       "Status": "{proposed_status}",
       "AI Advancement Recommendation": "{ai_recommendation}"
     }
   }
   ```

**REFLECT (Validate):**
1. Check API response: did update succeed?
   - If error: show error message to user
   - If date property error: verify format is `date:NAction Due:start` and `date:NAction Due:is_datetime: 0`
2. Sanity check: do updated values make sense?
   - Update Log has new entry at top
   - Biz Funnel didn't skip stages
   - Next Action is specific and actionable
3. Consider: should this trigger another agent? (e.g., stage change → meeting prep)
4. Log which interactions were analyzed (for debugging)

## Constraints
- Update Log MUST be ≤10 words
- NEVER skip Biz Funnel stages
- ALWAYS append to Update Log (never replace)
- "Waiting Feedback" status ONLY if truly waiting on client
- Next Action must be specific enough to know what to do
- Date properties use format: `date:NAction Due:start` and `date:NAction Due:is_datetime: 0`

## Output Format
```markdown
# Opportunity Update: [EXACT Opportunity Name from Notion]

> **Oportunidade:** [Exact opportunity name]
> **Notion URL:** [Direct link to opportunity page]

**IMPORTANT:** Use the EXACT opportunity name fetched from Notion (from `properties["Nome Oportunidade"]` or page title). This allows the user to immediately validate they are reviewing the correct opportunity.

**Last Interaction:** {meeting_date}
**Current Stage:** {current_biz_funnel} → {suggested_biz_funnel (if changed)}
**Priority:** {priority}

---

## Proposed Updates

### 1. Update Log
```
{proposed_update_log}
```
**Current (first 2-3 lines):**
```
{first 2-3 lines of current update log}
```

### 2. Next Action
```
{proposed_next_action}
```
**Current:** {current_next_action}

### 3. NAction Due
```
{proposed_naction_due}
```
**Current:** {current_naction_due}
**Rationale:** Last interaction + 7 days (or client deadline if mentioned)

### 4. Biz Funnel Stage
```
{proposed_biz_funnel}
```
**Current:** {current_biz_funnel}
**Changed:** {yes/no}
{if changed: **Reasoning:** {1 sentence why}}

### 5. Status
```
{proposed_status}
```
**Current:** {current_status}

### 6. AI Advancement Recommendation ⭐
```
{ai_recommendation (2-3 sentences with strategic insights)}
```
**Purpose:** Strategic insights and patterns across {N} recent interactions

---

## Meeting Context Analyzed

**Interactions analyzed:** {N} recent meetings/interactions
**Key patterns:** {extracted from multi-source analysis}
**Transcript excerpt:** {first 200 chars from most relevant interaction...}

---

**Apply these updates to Notion?** [Y/N]
```
