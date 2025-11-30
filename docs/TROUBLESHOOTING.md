# Troubleshooting Guide

Common issues and solutions for CRM workflow automation with Claude Code + Notion.

---

## Table of Contents

- [MCP Connection Issues](#mcp-connection-issues)
- [Opportunity Search Problems](#opportunity-search-problems)
- [Agent Execution Errors](#agent-execution-errors)
- [Update Failures](#update-failures)
- [Performance Issues](#performance-issues)
- [Known Issues & Patterns](#known-issues--patterns)

---

## MCP Connection Issues

### Problem: "MCP tool not available" or "mcp__notion__notion-search not found"

**Symptoms:**
- Agent reports MCP tools unavailable
- Natural language invocation fails
- Slash command works but agent can't access Notion

**Root Cause:**
MCP tools are conversation-level resources. When agents are invoked via subagent (Task tool), they execute in isolated context without MCP access.

**Solution:**
✅ **Use slash commands instead of natural language:**
```
/analyze-opportunity Musashi - 600k
```

Slash commands execute in main conversation context where MCP tools are guaranteed available.

**Technical Details:**
- See Principle 27 in PRINCIPLES.md
- See failure-log.md entry: "2025-11-29 Auto-Trigger Inconsistency & MCP Isolation"

---

### Problem: MCP Server Not Connecting on Startup

**Symptoms:**
- Claude Code opens but no MCP servers show as connected
- Error: "Failed to connect to Notion MCP server"

**Diagnostic Steps:**
1. **Check Node.js is installed:**
   ```bash
   node --version  # Should be >= 18.x
   ```

2. **Verify `.env` file exists:**
   ```bash
   ls -la .env  # macOS/Linux
   dir .env     # Windows
   ```

3. **Check API key is set:**
   ```bash
   cat .env | grep NOTION_API_KEY
   ```
   Should show: `NOTION_API_KEY=secret_xxxxxxxxxx`

4. **Test MCP server manually:**
   ```bash
   npx -y @notionhq/mcp-server-notion
   ```

**Solutions:**

**Missing Node.js:**
```bash
# macOS (Homebrew)
brew install node

# Windows
Download from https://nodejs.org

# Linux (apt)
sudo apt install nodejs npm
```

**Missing `.env` file:**
```bash
cp .env.example .env
# Edit .env and add your NOTION_API_KEY
```

**Invalid API Key:**
1. Go to https://www.notion.so/my-integrations
2. Create new integration or refresh existing key
3. Update `.env` with new key
4. Restart Claude Code

**Windows PATH Issues:**
- Ensure Node.js is in system PATH
- Restart terminal/IDE after installing Node.js

---

## Opportunity Search Problems

### Problem: "Opportunity not found" (but it exists in Notion)

**Symptoms:**
- Agent can't find opportunity by name
- Search returns no results or wrong results

**Common Causes & Solutions:**

**1. Case Sensitivity**
```
❌ /analyze-opportunity projeto sidi
✅ /analyze-opportunity Projeto SiDi - Nova Plataforma
```
Use exact name as shown in Notion (including capitalization).

**2. Partial Name**
```
❌ /analyze-opportunity Ferreira
✅ /analyze-opportunity Ferreira Costa - Sávio e Juarez Guimarães
```
Use complete opportunity name.

**3. Special Characters**
Some characters cause search issues. **Workaround:** Use Notion URL instead:
```
/analyze-opportunity https://www.notion.so/Ferreira-Costa-a4b96b2520374a358448c06eda4b4000
```

**4. Post-Search Filtering**
Agent applies post-search filtering to validate results. If search returns multiple matches, agent shows list and asks you to choose.

**Validation Protocol:**
1. Agent searches database
2. Shows result: name, URL, database
3. Asks: "Is this the correct opportunity? (Y/N)"
4. If N: requests correct URL from you

See `.claude/agents/opportunity-advancer.md` lines 35-53 for validation logic.

---

### Problem: Search Returns Wrong Opportunity

**Symptoms:**
- Search finds an opportunity but it's the wrong one
- Multiple opportunities with similar names

**Root Cause:**
Notion search ranking may prioritize recently modified or partial matches.

**Solution:**
1. Agent shows you the result before proceeding
2. Validate the Notion URL carefully
3. If wrong, answer `N` and provide correct URL
4. Or use exact URL from the start:
   ```
   /analyze-opportunity https://notion.so/exact-page-id
   ```

**Prevention:**
Use unique opportunity names in Notion (e.g., include company + project name).

---

## Agent Execution Errors

### Problem: "No meeting data to process"

**Symptoms:**
- Agent finds opportunity successfully
- But reports no interactions or meeting notes to analyze

**Root Cause:**
Opportunity has no content in:
- Anotações (related meeting note pages), OR
- Offline Notes field

**Solution:**
1. Add meeting notes as Anotação (related page)
2. OR populate Offline Notes field directly in opportunity
3. Verify Anotação contains content (not just empty page)

**Anotação Content Requirements:**
- Must have at least one of: `<transcript>`, `<summary>`, or `<notes>` tags
- Content inside tags must not be empty

---

### Problem: Agent Gives Vague Recommendations

**Symptoms:**
- "Follow up with client" without specifics
- Stage advancement without justification
- No concrete next steps

**Root Cause:**
Violation of Principle 5 (Be Explicit, Never Vague).

**Documented in:**
- failure-log.md: "2025-11-29 opportunity-advancer - Vague recommendations"

**Solutions Implemented:**
- Structured memory logging (`.claude/memory/opportunity-analysis-log.jsonl`)
- Enhanced PLAN phase with explicit patterns
- REFLECT phase validation checkpoint

**What to do:**
1. Reject vague recommendations (`N` or `E` to edit)
2. Provide feedback: "The recommendation is too vague, please be specific"
3. cs-agent-validator will capture feedback and create improvement issue

**Prevention:**
Agent now includes good vs bad recommendation examples in PLAN phase.

---

### Problem: Agent Suggests Skipping Biz Funnel Stages

**Symptoms:**
- Agent recommends jumping from stage 2 to stage 5
- Skips intermediate stages

**Business Rule:**
Biz Funnel stages should progress sequentially without skipping (Marketing → Suspect → Prospect → Contato → Credibilidade → Oferta → Proposta → Negociação → Fechamento → Relacionamento → done!).

**Solution:**
1. Agent now warns if advancement skips stages
2. Requires explicit confirmation for non-sequential moves
3. You can reject the update (`N`)

**Documented in:**
- CLAUDE.md: "Biz Funnel stages should not be skipped. Warn when doing so."
- `.claude/skills/crm-data-model/SKILL.md`: Stage definitions and advancement signals

**Override:**
If justified (e.g., client explicitly requested proposal without prior credibility building), you can approve with understanding that stage was intentionally skipped.

---

## Update Failures

### Problem: "Failed to update opportunity" or "Update request denied"

**Symptoms:**
- Agent proposes updates successfully
- You approve (`Y`)
- Update fails with error

**Common Causes:**

**1. Permission Denied**
- Notion MCP write operations require user approval
- Check `.claude/settings.local.json` - write operations not in auto-approve list
- **Expected:** Claude Code should prompt for approval
- **If not prompting:** Restart Claude Code

**2. Invalid API Key**
```bash
# Verify key is valid
cat .env | grep NOTION_API_KEY
```
- Regenerate key at https://www.notion.so/my-integrations if expired

**3. Network Issues**
- Check internet connection
- Notion API may be temporarily down (check https://status.notion.so)

**4. Field Validation**
- Some fields have constraints (e.g., Biz Funnel must be valid stage)
- Agent should validate before attempting update
- If validation missed, report as feedback

---

### Problem: Update Succeeded But Fields Look Wrong

**Symptoms:**
- Agent reports "✅ Updated successfully"
- But fields in Notion don't match what agent proposed

**Diagnostic:**
1. Check Notion page directly (use URL from agent output)
2. Verify which fields actually changed
3. Check Update Log for append vs replace behavior

**Common Issues:**

**Update Log Not Appending:**
- Agent should APPEND to Update Log, not replace
- Check if old entries were lost
- **If lost:** Bug in agent logic, report as feedback

**Wrong Date Format:**
- Next Action Date should be YYYY-MM-DD
- If showing as datetime, agent used wrong format
- Report as feedback

**Biz Funnel Not Changed:**
- Agent may have proposed change but didn't execute
- Check agent output logs for error during update

---

## Performance Issues

### Problem: Agent Takes Too Long (>30 seconds)

**Symptoms:**
- Agent execution exceeds expected time
- You're waiting >1 minute for results

**Expected Times:**
- Search + validation: 5-10 seconds
- Load interactions: 5-10 seconds
- Analysis + recommendations: 10-20 seconds
- **Total: 30-60 seconds typical**

**If exceeding 60 seconds:**

**1. Large Meeting Notes**
- Check if Anotações are very long (>5000 words)
- Agent loads 2-3 most recent interactions
- Long transcripts increase processing time

**2. Many Anotações**
- If opportunity has >10 related Anotações
- Agent still only loads 2-3 most recent
- But search/sort time increases

**3. Network Latency**
- MCP calls to Notion API depend on network
- Check internet connection speed

**Optimization:**
- Keep meeting notes concise (use summaries, not full transcripts)
- Archive old Anotações to separate database if >20 per opportunity

---

## Known Issues & Patterns

### From failure-log.md

**1. Slash Command Syntax Error (2025-11-29) - RESOLVED**
- Issue: Used `{{ARG}}` instead of `$ARGUMENTS`
- Solution: Corrected syntax
- Prevention: Always reference official Claude Code docs

**2. Auto-Trigger Inconsistency (2025-11-29) - RESOLVED**
- Issue: Natural language triggers unreliable, MCP unavailable in subagents
- Solution: Created `/analyze-opportunity` slash command
- Principle: 27 (Slash Commands for Deterministic Triggers)

**3. Vague Recommendations (2025-11-29) - MITIGATED**
- Issue: Agent produced non-actionable recommendations
- Solution: Added structured memory, enhanced PLAN phase
- Monitoring: Via `.claude/memory/opportunity-analysis-log.jsonl`

---

## When to Report Issues

**Use cs-agent-validator by providing feedback:**

Examples:
- "The agent skipped Biz Funnel stages again"
- "Search didn't find the opportunity even with exact name"
- "Agent took 5 minutes to respond"
- "Recommendations are still too vague"

**cs-agent-validator will:**
1. Capture your feedback
2. Validate against project specifications
3. Create GitHub issue if improvement needed
4. Track pattern if recurring

**Or create GitHub issue directly:**
- For bugs: Include exact error message, steps to reproduce
- For enhancements: Describe desired behavior
- Reference this troubleshooting guide in issue

---

## Getting More Help

**Documentation:**
- [QUICK-START.md](QUICK-START.md) - Common workflows
- [PRINCIPLES.md](PRINCIPLES.md) - System design philosophy
- [governance/failure-log.md](governance/failure-log.md) - All known issues
- [../README.md](../README.md) - Environment setup

**Configuration:**
- `.claude/agents/opportunity-advancer.md` - Agent logic
- `.claude/skills/crm-data-model/SKILL.md` - Business rules
- `.claude/settings.local.json` - Permissions

**Logs:**
- `.claude/memory/opportunity-search-log.jsonl` - Search validation history
- `.claude/memory/opportunity-analysis-log.jsonl` - Analysis outcomes

**Community:**
- File GitHub issue: https://github.com/rogeriorm/crm/issues
- Check existing issues for similar problems

---

## Emergency Recovery

### If Everything Breaks

1. **Restart Claude Code**
   - Close and reopen IDE
   - MCP servers reload on startup

2. **Verify Environment**
   ```bash
   node --version
   cat .env | grep NOTION_API_KEY
   ```

3. **Test MCP Manually**
   ```bash
   npx -y @notionhq/mcp-server-notion
   ```

4. **Check Git Status**
   ```bash
   git status
   git diff
   ```
   If recent changes broke something, revert:
   ```bash
   git restore <file>
   ```

5. **Last Resort: Fresh Clone**
   ```bash
   cd ..
   git clone <repo-url> crm-fresh
   cd crm-fresh
   cp ../crm/.env .
   ```

---

**Last Updated:** 2025-11-30
