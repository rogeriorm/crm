## Arquitetura Técnica Atual

### Visão Geral (Híbrida)

A arquitetura atual implementa uma **abordagem híbrida** onde cada agente existe em duas camadas:

```
┌──────────────────────────────────────────────────────────┐
│                    USER INTERFACE                         │
│  - Notion Workspace (data entry, visualization)          │
│  - Claude Code CLI (automation, analysis)                │
└───────────────┬──────────────────────────────────────────┘
                │
    ┌───────────┴────────────┐
    │                        │
    ▼                        ▼
┌─────────────┐      ┌──────────────────┐
│  NOTION AI  │      │  CLAUDE CODE     │
│  (Agents)   │      │  (Agent Mirrors) │
└──────┬──────┘      └────────┬─────────┘
       │                      │
       └──────────┬───────────┘
                  │
                  ▼
       ┌──────────────────────┐
       │   NOTION DATA LAYER  │
       │   (via MCP Protocol) │
       │                      │
       │  - Oportunidades DB  │
       │  - Anotações         │
       │  - Contatos          │
       │  - Empresas          │
       └──────────────────────┘
```

### Componentes

#### 1. Notion Data Layer
**Responsabilidade:** Source of truth para todos os dados de negócio

**Databases:**
- **Oportunidades** (`collection://201b1882-308d-4524-8a86-6672d5502299`)
  - Properties: Task, Biz Funnel, Status, Priority, Next Action, Next Action Date, Update Log, Days in Stage, Days Until Due, Due, Anotações (relation), Offline Notes
- **Anotações:** Meeting notes com transcripts, summaries, notes
- **Contatos:** People database
- **Cliente:** Companies database

**Acesso:** MCP Protocol via `@notionhq/mcp-server-notion`

#### 2. Notion AI Agents (Camada 1)
**Responsabilidade:** Execução nativa no Notion, configurada via My Notion AI

**Agentes Implementados:**
- **Process 2.2 - Opportunity Advancer** ✅
  - **Trigger:** `"Atualize o avanço da oportunidade [Nome]"`
  - **Workflow:**
    1. Load oportunidade por nome ou URL
    2. Load Anotações relacionadas (todas), identificar mais recente
    3. Parse `<transcript>`, `<summary>`, `<notes>` + Offline Notes
    4. Analyze e auto-preenche 5 campos:
       - Update Log (1 sentence, max 10 words)
       - Next Action (verb + object)
       - Next Action Date (last interaction + 7 days)
       - Biz Funnel (detect advancement signals)
       - Status (In Progress / Waiting Feedback / Scheduled)
    5. User reviews e confirma
  - **Configuração:** Fully documented in My Notion AI page

- **Update Log Consolidator** ✅
  - **Trigger:** `"Consolida updates"`
  - **Workflow:** Prepends current date to Update Log entries, maintains reverse chronological order

#### 3. Claude Code Agent Mirrors (Camada 2)
**Responsabilidade:** Versão automatizada/programática dos agentes Notion, com versionamento e controle

**Status:** 🚧 Em desenvolvimento

**Vantagens:**
- ✅ Versionamento em Git
- ✅ Execução via CLI/scripts
- ✅ Testing automatizado
- ✅ Logs detalhados
- ✅ Integração com CI/CD (futuro)
- ✅ Orquestração programática (futuro)

**Agentes Planejados:**
- **Process 2.2 - Opportunity Advancer Mirror** 🎯 PRIORIDADE 0
  - Mesma lógica do agente Notion
  - Execução: `claude-code advance-opportunity "Nome da Opp"`
  - Output: Markdown report + Notion update confirmation

- **Process 3.0 - Weekly Pipeline Digest** 🎯 PRIORIDADE 1
  - Query oportunidades ativas
  - Calcular métricas de saúde
  - Identificar riscos/oportunidades
  - Gerar digest acionável

#### 4. MCP Protocol Integration
**Responsabilidade:** Comunicação entre Claude Code e Notion

**Configuration:** `.mcp.json`
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

**Auto-approved Operations:**
- `mcp__notion__notion-search` - Search without user approval
- `mcp__notion__notion-fetch` - Fetch pages/databases without approval

**Requires Approval:**
- `mcp__notion__notion-create-pages`
- `mcp__notion__notion-update-page`
- `mcp__notion__notion-update-database`

### Data Flow: Process 2.2 (Current Implementation)

```
┌─────────────────────────────────────────────────────────────┐
│  1. USER: "Atualize o avanço da oportunidade Projeto X"     │
└────────────────────────┬────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────┐
│  2. NOTION AI: Parse command, extract "Projeto X"           │
└────────────────────────┬────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────┐
│  3. NOTION API: Query Oportunidades by "Task" = "Projeto X" │
└────────────────────────┬────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────┐
│  4. NOTION API: Load all pages in "Anotações" relation      │
│     - Sort by Date property                                  │
│     - Identify most recent                                   │
└────────────────────────┬────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────┐
│  5. AI PROCESSING: Analyze transcript + notes                │
│     - Extract key decisions, commitments, signals            │
│     - Detect Biz Funnel advancement (keywords, context)      │
│     - Generate concise Update Log entry                      │
│     - Determine appropriate Next Action                      │
│     - Calculate Next Action Date                             │
└────────────────────────┬────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────┐
│  6. NOTION AI: Present 5 fields for user review              │
│     [Show proposed changes in UI]                            │
└────────────────────────┬────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────┐
│  7. USER: Review and confirm (or adjust)                     │
└────────────────────────┬────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────┐
│  8. NOTION API: Update opportunity page with new values      │
│     ✅ Pipeline advanced, next action clear                  │
└─────────────────────────────────────────────────────────────┘
```

### Technology Stack

| Layer | Technology | Version |
|-------|-----------|---------|
| **Data Layer** | Notion | API v2022-06-28 |
| **Integration Protocol** | Model Context Protocol (MCP) | Latest |
| **MCP Server** | @notionhq/mcp-server-notion | Latest |
| **Automation Layer (Notion)** | My Notion AI | Native |
| **Automation Layer (Code)** | Claude Code | Latest |
| **Runtime** | Node.js | >= 18.x |
| **Version Control** | Git | Any |

### Security & Permissions

- **API Key Storage:** `.env` file (gitignored)
- **Notion Integration:** Read/Write access to specific databases only
- **MCP Auto-approval:** Read operations only (search, fetch)
- **Human-in-the-loop:** All write operations in Claude Code require explicit user approval

### Current Limitations

1. **No Event Triggers:** Agents run on-demand, não event-driven
2. **No Orchestration:** Each agent operates independently
3. **Manual Sync:** Notion AI e Claude Code agents não sincronizam automaticamente
4. **No Testing Framework:** Validação é manual
5. **No Logging:** Sem histórico de execuções ou auditing

### Next Evolution: Agent Mirrors

**Goal:** Create Claude Code mirrors of Notion AI agents

**Benefits:**
- Test new agent logic before deploying to Notion AI
- Version control for agent behavior
- Automated testing and validation
- Detailed execution logs
- Foundation for orchestration layer

**Implementation Strategy:**
1. Start with Process 2.2 mirror (validate approach)
2. Add Weekly Pipeline Digest (net-new agent)
3. Gradually mirror other processes
4. Eventually: Orchestration layer coordinates both Notion AI and Claude Code agents

---

## Arquitetura Técnica Futura

### Visão de Orquestração Integrada

```
┌─────────────────────────────────────────────────────────────┐
│                    CRM ORCHESTRATOR                          │
│  (Event-driven coordinator - executa workflows integrados)  │
└──────────────────────┬──────────────────────────────────────┘
                       │
         ┌─────────────┼─────────────┬─────────────┐
         │             │             │             │
         ▼             ▼             ▼             ▼
┌─────────────┐ ┌─────────────┐ ┌──────────┐ ┌─────────────┐
│ Opportunity │ │  Meeting    │ │ Pipeline │ │  Follow-up  │
│   Creator   │ │   Prep      │ │  Digest  │ │   Drafter   │
│   (1.0)     │ │  Briefer    │ │  (3.0)   │ │   (4.0)     │
│             │ │   (2.1)     │ │          │ │             │
└──────┬──────┘ └──────┬──────┘ └────┬─────┘ └──────┬──────┘
       │               │              │              │
       └───────────────┴──────────────┴──────────────┘
                       │
                       ▼
         ┌─────────────────────────┐
         │  Opportunity Advancer   │
         │       (2.2) ✅          │
         │  (já implementado!)     │
         └─────────────────────────┘
                       │
                       ▼
         ┌─────────────────────────┐
         │   NOTION DATA LAYER     │
         │  (via MCP Protocol)     │
         │ - Oportunidades DB      │
         │ - Anotações (notes)     │
         │ - Contatos/Empresas     │
         └─────────────────────────┘
```

### Exemplo de Workflow Orquestrado

**Trigger:** Nova página criada em "Anotações" (reunião com cliente)

**Workflow:**
1. **Orchestrator detecta:** Anotação linkada à Oportunidade X
2. **Chama Agent 2.2 (Opportunity Advancer):**
   - Processa transcript
   - Preenche 5 campos
   - Detecta: Biz Funnel avançou de Credibilidade → Proposta
3. **Orchestrator detecta:** Stage advancement = evento relevante
4. **Chama Agent 2.1 (Meeting Prep Briefer):**
   - Verifica se há próxima reunião agendada (Next Action = "Apresentar proposta")
   - Gera briefing proativo para a apresentação
5. **Orchestrator detecta:** Briefing gerado, oportunidade em Proposta
6. **Chama Agent 4.0 (Follow-up Drafter) - OPCIONAL:**
   - Gera draft de thank-you email pós-reunião
   - Inclui recap de decisões e next steps

**Resultado:** Pipeline avançado + próxima reunião preparada + follow-up rascunhado, tudo em <30 segundos, zero esforço manual.