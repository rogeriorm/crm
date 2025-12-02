---
status: Current
version: 1.0
last_updated: 2025-12-01
language: en
audience: developers
---

# MCP Tools Reference

Reference guide for Notion MCP tools available in Claude Code.

**Protocol:** Model Context Protocol (MCP)
**Server:** `@notionhq/mcp-server-notion` (auto-loaded via npx)
**Configuration:** `.mcp.json` + `.env` (NOTION_API_KEY)
**Permissions:** `.claude/settings.local.json`

See [MCP Integration Architecture](./diagrams/mcp-integration.mmd) for visual overview.

---

## Auto-Approved Tools (Read-Only)

These tools execute without user confirmation:

### `mcp__notion__notion-search`

Semantic search across Notion workspace and connected sources.

**Parameters:**
- `query` (required): Search string
- `query_type`: `"internal"` (default) or `"user"`
- `page_url`: Restrict search to page/subpages
- `data_source_url`: Search within database (use `collection://` URL)
- `teamspace_id`: Restrict to teamspace
- `filters`: Date range and creator filters

**Example:**
```typescript
{
  "query": "Musashi - 600k",
  "data_source_url": "collection://201b1882-308d-4524-8a86-6672d5502299"
}
```

**Returns:** Search results with page URLs and snippets

**Use Cases:**
- Find opportunity by name in Oportunidades database
- Search meeting notes (Anotações)
- Validate user-provided opportunity names

**Error Handling:**
- No results → Ask user to verify spelling or provide Notion URL
- Multiple results → Show all, ask user to select
- MCP unavailable → Fallback to asking user for Notion URL

---

### `mcp__notion__notion-fetch`

Retrieve page or database details by URL or ID.

**Parameters:**
- `id` (required): Notion URL or page/database ID

**Example:**
```typescript
{
  "id": "https://notion.so/workspace/Musashi-600k-abc123"
}
```

**Returns:**
- **Pages:** Properties + content in Notion-flavored Markdown
- **Databases:** Schema + all data sources

**Use Cases:**
- Load opportunity properties (Biz Funnel, Status, Next Action, etc.)
- Fetch recent meeting notes for analysis
- Retrieve database schema for validation

**Error Handling:**
- Invalid ID → Verify URL format or ask user
- Permission denied → Check Notion integration access
- Page not found → Confirm page wasn't deleted

---

## User-Approval Required (Write Operations)

These tools require explicit user confirmation:

### `mcp__notion__notion-update-page`

Update page properties or content.

**Parameters:**
- `page_id` (required): Page ID (with or without dashes)
- `data.command`: `"update_properties"`, `"replace_content"`, `"replace_content_range"`, `"insert_content_after"`
- `data.properties`: Property updates (for `update_properties`)
- `data.new_str`: New content
- `data.selection_with_ellipsis`: Target range (for partial updates)

**Property Format Examples:**

**Text/Title:**
```typescript
{
  "properties": {
    "Next Action": "Send proposal draft for review"
  }
}
```

**Date (always split into 3 fields):**
```typescript
{
  "properties": {
    "date:Next Action Date:start": "2025-12-15",
    "date:Next Action Date:end": null,
    "date:Next Action Date:is_datetime": 0
  }
}
```

**Select:**
```typescript
{
  "properties": {
    "Biz Funnel": "Oferta",
    "Status": "In Progress"
  }
}
```

**Number:**
```typescript
{
  "properties": {
    "Priority": 5
  }
}
```

**Checkbox:**
```typescript
{
  "properties": {
    "Is Active": "__YES__"  // or "__NO__"
  }
}
```

**Multi-Select:**
```typescript
{
  "properties": {
    "Tags": "Enterprise, High-Value, Q4"
  }
}
```

**CRITICAL: Update Log Format (Append Only)**

Never replace Update Log. Always append with timestamp:

```typescript
{
  "command": "replace_content_range",
  "selection_with_ellipsis": "# Update Log...last entry text",
  "new_str": "# Update Log\n\n## 2025-12-01 14:30 #ai\nUpdated Biz Funnel from Contato to Credibilidade based on 2 positive meetings...\n\n[previous entries]"
}
```

**Use Cases:**
- Update 6 opportunity fields (opportunity-advancer workflow)
- Append AI recommendations to Update Log
- Update Next Action and dates based on meeting analysis

**Error Handling:**
- Property not found → Check database schema with fetch first
- Invalid date format → Use YYYY-MM-DD for dates
- Rate limit (3 req/sec) → Batch updates in single call
- Validation error → Show error, ask user to edit proposed values

---

### `mcp__notion__notion-create-pages`

Create one or more Notion pages.

**Parameters:**
- `pages` (required): Array of page objects
- `parent`: `page_id`, `database_id`, or `data_source_id`

**Example:**
```typescript
{
  "parent": {
    "data_source_id": "201b1882-308d-4524-8a86-6672d5502299"
  },
  "pages": [{
    "properties": {
      "Name": "New Opportunity - Acme Corp",
      "Biz Funnel": "Suspect",
      "Status": "In Progress"
    },
    "content": "# Initial Notes\n\nFirst contact via referral..."
  }]
}
```

**Use Cases:**
- Create new opportunities (future workflow)
- Create meeting notes from transcripts (future)

**Error Handling:**
- Missing title property → Add Name property (required for databases)
- Invalid parent → Verify data_source_id from fetch

---

### `mcp__notion__notion-move-pages`

Move pages or databases to new parent.

**Parameters:**
- `page_or_database_ids`: Array of IDs to move
- `new_parent`: Target page/database/workspace

**Use Cases:**
- Reorganize workspace structure
- Archive old opportunities

---

## Configuration Files

### `.mcp.json`

MCP server configuration (auto-loads on Claude Code startup):

```json
{
  "mcpServers": {
    "notion": {
      "command": "npx",
      "args": ["-y", "@notionhq/mcp-server-notion"],
      "env": {
        "NOTION_API_KEY": "${NOTION_API_KEY}"
      }
    }
  }
}
```

### `.env`

API credentials (NOT committed to git):

```bash
NOTION_API_KEY=secret_abc123...
```

### `.claude/settings.local.json`

Tool permissions:

```json
{
  "allowedCommands": ["mcp__notion__notion-search", "mcp__notion__notion-fetch"]
}
```

**Security Model:**
- Read operations (search, fetch) → Auto-approved
- Write operations (update, create, move) → User approval required
- API key never exposed to conversation

---

## Rate Limits

**Notion API:** 3 requests/second

**Best Practices:**
- Batch property updates in single `update-page` call (not 6 separate calls)
- Use search filters to narrow results
- Cache fetch results instead of re-fetching

---

## Common Patterns

### Pattern: Search → Validate → Fetch → Update

```typescript
// 1. SENSE: Search for opportunity
const searchResult = await mcp__notion__notion-search({
  query: "Musashi - 600k",
  data_source_url: "collection://201b1882-308d-4524-8a86-6672d5502299"
});

// 2. SENSE: Validate with user
// "Is this correct? [Musashi - 600k](https://notion.so/...) (Y/N)"

// 3. SENSE: Fetch full data
const opportunity = await mcp__notion__notion-fetch({
  id: searchResult.url
});

// 4. PLAN: Analyze patterns, generate recommendations

// 5. ACT: Update (requires approval)
await mcp__notion__notion-update-page({
  page_id: opportunity.id,
  data: {
    command: "update_properties",
    properties: {
      "Biz Funnel": "Oferta",
      "Status": "In Progress",
      "Next Action": "Send proposal",
      "date:Next Action Date:start": "2025-12-15",
      "date:Next Action Date:is_datetime": 0,
      "AI Recommendation": "Strong buying signals..."
    }
  }
});

// 6. REFLECT: Log to memory (JSONL)
```

### Pattern: Append to Update Log

```typescript
// 1. Fetch current Update Log content
const page = await mcp__notion__notion-fetch({ id: opportunityUrl });
const currentLog = extractUpdateLog(page.content);

// 2. Find unique snippet for replacement
const snippet = currentLog.substring(0, 20) + "..." +
                currentLog.substring(currentLog.length - 20);

// 3. Prepend new entry
const newLog = `# Update Log\n\n## ${timestamp} #ai\n${newEntry}\n\n${currentLog}`;

// 4. Update with replace_content_range
await mcp__notion__notion-update-page({
  page_id: opportunityId,
  data: {
    command: "replace_content_range",
    selection_with_ellipsis: snippet,
    new_str: newLog
  }
});
```

---

## Troubleshooting

See [TROUBLESHOOTING.md](../TROUBLESHOOTING.md) for detailed MCP issues:

**Problem:** "MCP tool not available"
**Solution:** Use slash commands instead of natural language

**Problem:** Search returns no results
**Solution:** Verify database URL, check spelling, try broader query

**Problem:** Date property update fails
**Solution:** Use split format: `date:PropertyName:start`, `date:PropertyName:is_datetime`

**Problem:** Rate limit errors
**Solution:** Batch updates, reduce parallel calls

---

## Related Documentation

- **Architecture:** [overview.md](./overview.md) - System design and MCP integration
- **Visual:** [diagrams/mcp-integration.mmd](./diagrams/mcp-integration.mmd) - MCP protocol flow
- **Workflow:** [diagrams/agent-workflow.mmd](./diagrams/agent-workflow.mmd) - SPAR sequence
- **Principles:** [../PRINCIPLES.md](../PRINCIPLES.md) - Principle 13 (Show Before Execute)
- **Agent Spec:** [../agents/opportunity-advancer-spec.md](../agents/opportunity-advancer-spec.md) - Real usage example
