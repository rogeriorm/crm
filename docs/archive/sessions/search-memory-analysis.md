# Search Memory Analysis System

**Version:** 1.0
**Created:** 2025-11-23
**Purpose:** Track opportunity search accuracy to guide future improvements

---

## Overview

The opportunity-advancer agent logs every search interaction to build a memory of search accuracy patterns. This enables data-driven improvements to search logic over time.

**File Location:** `.claude/memory/opportunity-search-log.jsonl`

**Format:** JSONL (JSON Lines) - one JSON object per line, append-only

---

## File Schema

```json
{
  "timestamp": "ISO-8601 datetime string",
  "query": "user's search term",
  "returned_name": "opportunity name from search result",
  "returned_url": "notion URL from search result",
  "user_validated": boolean,
  "correction_needed": boolean,
  "correct_url": "string or null",
  "notes": "string or null"
}
```

### Field Descriptions

| Field | Type | Description |
|-------|------|-------------|
| `timestamp` | string | ISO-8601 format (YYYY-MM-DDTHH:MM:SS) |
| `query` | string | The search term user provided |
| `returned_name` | string | Opportunity name from search result |
| `returned_url` | string | Full Notion URL from search result |
| `user_validated` | boolean | `true` if user confirmed correct, `false` if rejected |
| `correction_needed` | boolean | `true` if user provided different URL, `false` otherwise |
| `correct_url` | string\|null | User-provided URL (only if `correction_needed=true`) |
| `notes` | string\|null | Optional context about the search |

---

## Example Entries

### Successful Match (User Confirmed)
```json
{"timestamp": "2025-11-23T14:30:00", "query": "Ferreira Costa", "returned_name": "Ferreira Costa - Savio e Juarez Guimarães", "returned_url": "https://www.notion.so/rogeriormoreira/Ferreira-Costa-Savio-e-Juarez-Guimar-es-a4b96b2520374a358448c06eda4b4000", "user_validated": true, "correction_needed": false, "correct_url": null, "notes": null}
```

### Failed Match (User Rejected)
```json
{"timestamp": "2025-11-23T15:45:00", "query": "Ferreira Costa", "returned_name": "Ferreira Costa - Wrong Contact", "returned_url": "https://www.notion.so/Ferreira-Costa-146b1882308d80a9b0b5d2d7bb0f15", "user_validated": false, "correction_needed": true, "correct_url": "https://www.notion.so/rogeriormoreira/Ferreira-Costa-Savio-e-Juarez-Guimar-es-a4b96b2520374a358448c06eda4b4000", "notes": "User rejected search result"}
```

---

## Analysis Queries

### Basic Statistics

**Count total searches:**
```bash
wc -l < .claude/memory/opportunity-search-log.jsonl
```

**Success rate:**
```bash
# Total validated
grep '"user_validated":true' .claude/memory/opportunity-search-log.jsonl | wc -l

# Total corrections needed
grep '"correction_needed":true' .claude/memory/opportunity-search-log.jsonl | wc -l
```

### Using jq for Detailed Analysis

**Most problematic search terms:**
```bash
cat .claude/memory/opportunity-search-log.jsonl | \
  grep -v '^#' | \
  jq -r 'select(.correction_needed == true) | .query' | \
  sort | uniq -c | sort -rn
```

**Success rate by query:**
```bash
cat .claude/memory/opportunity-search-log.jsonl | \
  grep -v '^#' | \
  jq -r '[.query, (.user_validated | tostring)] | @csv' | \
  sort | uniq -c
```

**Recent failures (last 10):**
```bash
cat .claude/memory/opportunity-search-log.jsonl | \
  grep -v '^#' | \
  jq 'select(.correction_needed == true)' | \
  tail -10
```

**Average success rate over time:**
```bash
cat .claude/memory/opportunity-search-log.jsonl | \
  grep -v '^#' | \
  jq -r '[.timestamp[0:10], (.user_validated | if . then 1 else 0 end)] | @csv' | \
  awk -F, '{date[$1]+=$2; count[$1]++} END {for (d in date) print d, date[d]/count[d]*100"%"}'
```

---

## Patterns to Watch For

### 1. Partial Name Matches

**Problem:** User searches "Ferreira Costa" but multiple opportunities contain that phrase.

**Detection:**
```bash
# Find queries that frequently need correction
cat .claude/memory/opportunity-search-log.jsonl | \
  grep -v '^#' | \
  jq -r 'select(.correction_needed == true) | .query' | \
  sort | uniq -c | awk '$1 > 2 {print}'
```

**Action:** Consider prompting user to be more specific or showing top N results instead of just first.

### 2. Wrong Database Results

**Problem:** Search returns results from outside Oportunidades database.

**Detection:**
```bash
# Check if returned URLs don't match expected collection pattern
cat .claude/memory/opportunity-search-log.jsonl | \
  grep -v '^#' | \
  jq 'select(.correction_needed == true and (.returned_url | contains("rogeriormoreira") | not))'
```

**Action:** Verify `data_source_url` parameter in search is properly scoped.

### 3. Ranking Issues

**Problem:** Correct opportunity exists but isn't ranked first.

**Detection:** Compare `returned_name` with actual opportunity name from `correct_url`.

**Action:** Investigate MCP search ranking parameters or consider fuzzy matching.

---

## Review Cadence

**Weekly Review:**
- Check success rate (target: >85%)
- Identify top 3 problematic queries
- Review any new failure patterns

**Monthly Review:**
- Analyze trends over time
- Evaluate if improvements are needed
- Update search strategy if patterns emerge

**Ad-hoc Review:**
- After any search logic changes
- When user reports search issues
- Before implementing search optimizations

---

## Actionable Insights

### When Success Rate < 80%

**Investigate:**
1. Are partial names causing issues? → Prompt for full names
2. Are results from wrong database? → Fix scope parameter
3. Is ranking wrong? → Review MCP search parameters

**Implement:**
- Add fuzzy matching logic
- Show top 3 results instead of 1
- Improve search query preprocessing (e.g., normalize spacing)

### When Specific Query Fails Repeatedly

**Example:** "Ferreira Costa" fails 3+ times

**Action:**
1. Create test case for this specific query
2. Investigate why MCP search returns wrong result
3. Consider creating search hints/aliases
4. Document as known issue if unfixable at MCP level

---

## Privacy & Data Management

### What Gets Logged

- ✅ Search terms (opportunity names)
- ✅ URLs (public Notion page identifiers)
- ✅ Validation results (boolean flags)

### What Doesn't Get Logged

- ❌ Opportunity content (meeting notes, etc.)
- ❌ Personal customer data
- ❌ Financial information
- ❌ Sensitive business details

### Data Retention

**Current Policy:** Keep all historical data (learning value)

**Future Considerations:**
- Archive logs older than 1 year
- Anonymize after N months if privacy concerns arise
- GDPR compliance if working with EU customers

### Git Tracking

**Recommended:** Commit memory file to git for:
- Transparency
- Team collaboration
- Historical analysis
- Backup

**Alternative:** Add to .gitignore if privacy concerns exist

---

## Integration with Improvement Cycle

```
User Search → Validation → Log Entry
                              ↓
                    Weekly Analysis
                              ↓
                    Pattern Detection
                              ↓
                  GitHub Issue Created
                              ↓
              business-architect designs fix
                              ↓
              system-implementer applies fix
                              ↓
                    Monitor success rate
                              ↓
                      (cycle repeats)
```

---

## Success Criteria

**Excellent Performance:**
- ✅ Success rate > 90%
- ✅ No repeated failures for same query
- ✅ Consistent improvement trend over time

**Needs Improvement:**
- ⚠️ Success rate 70-85%
- ⚠️ Same query fails 2-3 times
- ⚠️ Flat or declining trend

**Critical Issues:**
- ❌ Success rate < 70%
- ❌ Same query fails 4+ times
- ❌ User bypasses search, provides URLs directly

---

## Example: From Memory to Action

### Step 1: Detect Pattern (Week 3 Review)
```bash
# Query fails 4 times
grep '"query":"Ferreira Costa"' .claude/memory/opportunity-search-log.jsonl | \
  jq 'select(.correction_needed == true)' | wc -l
# Output: 4
```

### Step 2: Analyze Root Cause
- Search returns older opportunity with same company name
- MCP sorts by relevance, not Last Edited Time
- User always wants most recent

### Step 3: Create Issue
**Title:** "Improve search ranking for duplicate company names"

**Body:**
```markdown
## Problem
"Ferreira Costa" query returns older opportunity, not most recent.

## Evidence
4 corrections logged in memory file (grep '"query":"Ferreira Costa"')

## Proposed Solution
Add Last Edited Time as secondary sort after relevance score.

## Expected Impact
Reduce correction_needed rate for partial name searches by ~30%.
```

### Step 4: Implement & Validate
- Update opportunity-advancer search logic
- Test with "Ferreira Costa" query
- Monitor next 10 searches for improvement

---

## References

- `.claude/agents/opportunity-advancer.md` - Agent implementation
- `docs/PRINCIPLES.md` - Principle 12 (Always Validate Against Source)
- `docs/agentic-workflow-map.md` - Process flow including validation step
- GitHub Issue #5 - Original bug that motivated this system

---

**Philosophy:** Every search is a learning opportunity. Track what works, fix what doesn't, improve continuously based on evidence, not assumptions.
