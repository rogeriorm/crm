## Arquitetura Técnica Atual

### Visão Geral (Híbrida)

**Visual Overview:** See [diagrams/system-overview.mmd](diagrams/system-overview.mmd) for three-layer architecture diagram.

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

**Visual:** See [diagrams/mcp-integration.mmd](diagrams/mcp-integration.mmd) for protocol flow diagram.
**Reference:** See [mcp-tools-reference.md](mcp-tools-reference.md) for complete tool documentation.

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

**Visual:** See [diagrams/data-flow.mmd](diagrams/data-flow.mmd) and [diagrams/agent-workflow.mmd](diagrams/agent-workflow.mmd) for complete SPAR workflow.

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
| **Governance Layer** | Progressive (Phase 0→3) | Evolving |
| **Runtime** | Node.js | >= 18.x |
| **Version Control** | Git | Any |

### Agent Governance Layer (Cross-Cutting)

**Status:** Phase 0 (Lightweight Practices)
**Architecture:** Cross-cutting concern that intersects SPAR at multiple points

**Visual:** See [diagrams/governance-progression.mmd](diagrams/governance-progression.mmd) for Phase 0-3 evolution.

```
┌─────────────────────────────────────────────────────────────┐
│              AGENT GOVERNANCE LAYER                         │
│  (Cross-cutting: Audit, Control, Validation, Learning)     │
└─────────────────────────────────────────────────────────────┘
                           │
        ┌──────────────────┼──────────────────┐
        │                  │                  │
        ↓                  ↓                  ↓
   SENSE Phase       PLAN Phase         ACT Phase        REFLECT Phase
        │                  │                  │                  │
   Schema            Business Rule      Rate Limiting      Failure Registry
   Validation        Validation         Audit Logging      Rollback Capability
                                        User Approval      Learning
```

**Current Implementation (Phase 0):**
- ✅ Human approval gate (all writes require confirmation)
- ✅ Update Log field (implicit audit trail in Notion)
- ✅ CLAUDE.md constraints (documented rules)
- ✅ SPAR framework (built-in validation phases)
- ✅ Failure log (`docs/governance/failure-log.md`) - IMPLEMENTED

**Progressive Evolution:**
| Phase | Complexity | When | What |
|-------|-----------|------|------|
| **Phase 0** | Lightweight practices | Now (1 agent) | Human approval + docs |
| **Phase 1** | Shared conventions | 3+ agents | `/docs/governance/` docs |
| **Phase 2** | Reusable subsystem | 5+ agents | `/.claude/governance/` code |
| **Phase 3** | Dedicated service | Orchestration | `/services/governance/` APIs |

**Governance Capabilities by Phase:**

```
Phase 0 (NOW):
  ├─ Audit: Existing Update Log field
  ├─ Rate Limiting: Human approval gate
  ├─ Schema Validation: CLAUDE.md documentation
  ├─ Rollback: Manual (screenshot before changes)
  ├─ Usage Monitoring: Claude Code token display
  └─ Failure Learning: docs/governance/failure-log.md

Phase 1 (3+ agents):
  ├─ Audit: Standardized format conventions
  ├─ Rate Limiting: Documented operational limits
  ├─ Schema Validation: schema-reference.json
  ├─ Rollback: Still manual (infrequent need)
  ├─ Usage Monitoring: Weekly manual review
  └─ Failure Learning: Pattern extraction from log

Phase 2 (5+ agents):
  ├─ Audit: audit-logger.js (centralized)
  ├─ Rate Limiting: rate-limiter.js (shared logic)
  ├─ Schema Validation: schema-validator.js (pre-update checks)
  ├─ Rollback: rollback-manager.js (undo capability)
  ├─ Usage Monitoring: usage-tracker.js (automated)
  └─ Failure Learning: pattern-analyzer.js

Phase 3 (Complex orchestration):
  ├─ Audit: audit-service (centralized log service)
  ├─ Rate Limiting: policy-engine (auto-approve rules)
  ├─ Schema Validation: validation-service (real-time)
  ├─ Rollback: rollback-service (transaction-like)
  ├─ Usage Monitoring: monitoring-dashboard (real-time)
  └─ Failure Learning: ml-pattern-detection
```

**Key Principle:** "Earn Your Infrastructure"
- Don't build ahead of need
- Evidence-driven progression (failure patterns, volume, stakes)
- Governance emerges from complexity, not before it

**Signals to Advance Phase:**
| Signal | Threshold | Action |
|--------|-----------|--------|
| Same failure 3x | Pattern emerged | Build targeted prevention |
| Recovery > 1 hour | Stakes too high | Build rollback capability |
| Approval fatigue | >20 approvals/session | Build selective auto-approve |
| Cost surprise | Bill > 2x expected | Build usage monitoring |
| 3+ agents operational | Shared needs | Move to Phase 1 |
| 5+ agents operational | Volume justifies | Move to Phase 2 |

### Security & Permissions

- **API Key Storage:** `.env` file (gitignored)
- **Notion Integration:** Read/Write access to specific databases only
- **MCP Auto-approval:** Read operations only (search, fetch)
- **Human-in-the-loop:** All write operations in Claude Code require explicit user approval
- **Governance Posture:** Phase 0 (human approval IS the governance layer)

### Current Limitations

1. **No Event Triggers:** Agents run on-demand, não event-driven
2. **No Orchestration:** Each agent operates independently
3. **Manual Sync:** Notion AI e Claude Code agents não sincronizam automaticamente
4. **No Testing Framework:** Validação é manual
5. **Lightweight Governance:** Phase 0 practices (sufficient for current scale)

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