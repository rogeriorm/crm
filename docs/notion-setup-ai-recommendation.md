# Notion Setup: AI Advancement Recommendation Field

## Overview

The enhanced CRM skill now includes a 6th field called **AI Advancement Recommendation** that provides strategic insights by analyzing patterns across multiple customer interactions.

**Purpose:** Provide consultative insights that help identify:
- Pattern recognition (cliente mentioned X 3 times)
- Opportunity detection (ideal moment for proposal)
- Risk warnings (stalling signals, competitive threats)
- Blind spots (validate budget/authority before investing)

## Required Action

You need to manually add this new property to your Oportunidades database in Notion.

---

## Step-by-Step Instructions

### 1. Open Oportunidades Database

Navigate to your Oportunidades database in Notion:
- URL: `https://www.notion.so/[your-workspace]/201b1882308d45248a866672d5502299`
- Or search for "Oportunidades" in your workspace

### 2. Add New Property

1. **Open database view** (table, board, or any view)
2. **Click the "+" button** in the properties bar (top right of the database)
3. Or click on any column header → "+" at the end of properties list

### 3. Configure Property

**Property Name:**
```
AI Advancement Recommendation
```

**Property Type:**
```
Text
```

**Property Description (optional but recommended):**
```
Strategic insights and patterns from AI analysis across multiple customer interactions. Highlights opportunities, risks, and blind spots user might miss.
```

### 4. Position (Optional)

Recommended position: After "Next Action Date" field, before "Biz Funnel"

Order suggested:
1. Task (title)
2. Biz Funnel
3. Status
4. Priority
5. Update Log
6. Next Action
7. Next Action Date
8. **AI Advancement Recommendation** ← NEW
9. Anotações
10. Contatos
11. Cliente
12. Days in Stage
13. Days Until Due
14. Due

### 5. Verify Setup

After adding, verify the property:
- ✅ Name is exactly: `AI Advancement Recommendation`
- ✅ Type is: `Text`
- ✅ Property appears in all views of Oportunidades database

---

## Property Specifications

| Attribute | Value |
|-----------|-------|
| **Name** | AI Advancement Recommendation |
| **Type** | Text (not Rich Text) |
| **Required** | No (can be empty) |
| **Format** | 2-3 sentences with strategic insights |
| **Updated by** | opportunity-advancer agent (Claude Code) |
| **Frequency** | After each meeting/interaction analysis |

---

## Example Values

Once the agent starts populating this field, you'll see insights like:

**Example 1:**
```
Cliente mencionou orçamento em 3 interações recentes e agora confirmou aprovação -
momento ideal para proposta formal detalhada. Considere incluir case study similar
para reforçar credibilidade e acelerar decisão.
```

**Example 2:**
```
Sem movimento há 21 dias mas cliente mantém interesse (respondeu última mensagem
rapidamente). Sugerir reunião objetiva para validar timeline e desbloquear processo.
```

**Example 3:**
```
Padrão detectado: cliente está avaliando 2 fornecedores e pediu proposta de ambos.
Destacar diferenciais técnicos e referências do setor na proposta para criar vantagem.
```

**Example 4 (empty is OK):**
```
[empty field when no significant insights to add]
```

---

## Testing the Setup

After adding the property, test with the opportunity-advancer agent:

```bash
# In Claude Code, analyze an opportunity
# The agent should now output 6 fields instead of 5
```

The output should include:
1. Update Log
2. Next Action
3. Next Action Date
4. Biz Funnel
5. Status
6. **AI Advancement Recommendation** ← NEW

---

## Troubleshooting

### Issue: Property not found by agent

**Solution:**
- Verify property name is EXACTLY: `AI Advancement Recommendation` (case-sensitive)
- Check property type is `Text` (not `Rich Text` or other types)
- Refresh Notion page and try again

### Issue: Agent skips the field

**Solution:**
- This is OK if there are no significant insights to add
- The field can be empty - it's optional based on analysis

### Issue: Can't add property to database

**Solution:**
- Verify you have edit permissions for Oportunidades database
- Try adding from different view (table view usually works best)
- Check if database is locked or has restrictions

---

## Migration Notes

**Existing Opportunities:**
- Will have empty AI Advancement Recommendation field (this is normal)
- Field will be populated as you run opportunity-advancer agent on each opportunity
- No retroactive analysis is done automatically

**Notion AI Process 2.2:**
- Your existing Notion AI workflow continues to work with 5 fields
- Claude Code version adds the 6th field as enhancement
- Both can coexist without conflict

---

## Next Steps

After adding the property:

1. ✅ Verify property exists in Oportunidades database
2. ✅ Test with one opportunity using opportunity-advancer agent
3. ✅ Review AI recommendations for quality and usefulness
4. ✅ Provide feedback to refine recommendation logic if needed

---

## Questions?

If you encounter issues:
1. Check property name spelling (case-sensitive)
2. Verify property type is `Text`
3. Confirm you have edit permissions
4. Try in different browser/device if Notion is acting up
5. Report issues at: https://github.com/anthropics/claude-code/issues
