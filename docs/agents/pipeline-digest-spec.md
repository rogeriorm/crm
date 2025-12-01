---

### Decisões Táticas (Weekly Pipeline Digest)

4. **Frequência de Execução:**
   - [ ] Automático toda segunda 9:00 AM (requer trigger agendado - complexo)
   - [ ] Sob demanda via comando "gere pipeline digest" (simples, flexível)
   - [ ] Híbrido: lembrete automático segunda 9AM, execução sob demanda

5. **Formato de Entrega:**
   - [ ] Criar página no Notion "Pipeline Digests" (histórico rastreável)
   - [ ] Retornar markdown no chat (rápido, sem persistência)
   - [ ] Ambos: página Notion + notificação no chat

6. **Nível de Detalhe:**
   - [ ] Digest compacto: apenas action items + métricas críticas (1 página)
   - [ ] Digest completo: tudo descrito acima (2-3 páginas)
   - [ ] Digest adaptativo: compacto se tudo OK, completo se houver riscos
   

## 3. Especificação: Weekly Pipeline Digest (Prioridade 1)

### Objetivos

1. **Eliminar revisão manual de views** (30 min/semana → 5 min)
2. **Proatividade:** Identificar problemas antes de virarem crises
3. **Acionabilidade:** Output = lista clara de próximas ações
4. **Consistência:** Executar toda segunda-feira 9:00 AM (ou sob demanda)

### Inputs

**Data Sources:**
- Oportunidades database: `collection://201b1882-308d-4524-8a86-6672d5502299`
- Views existentes (queries):
  - 🚨 Pipeline Travado: `Days in Stage > 14`
  - ⏳ Aguardando Cliente: `Status = "Waiting Feedback"`
  - 📅 Esta Semana: `Next Action Date in next 7 days OR overdue`
  - 💰 Active Pipeline: Stages de venda ativos

**Parâmetros (opcionais):**
- `date_range`: Análise semanal (default) ou custom
- `focus`: "risks" | "revenue" | "full" (default)
- `format`: "markdown" | "bullets" | "email"

### Processamento (Lógica do Agent)

**Step 1: Coletar Dados**
```
- Query Oportunidades (todas ativas, exclude done/archived)
- Para cada opp, load properties:
  - Biz Funnel, Status, Next Action, Next Action Date
  - Days in Stage, Days Until Due, Priority
  - Update Log (última entrada), Cliente, Contatos
```

**Step 2: Calcular Métricas de Saúde**
```
Pipeline Velocity:
  - Avg Days in Stage por Biz Funnel stage
  - Benchmark: <10 dias = healthy, 10-14 = warning, >14 = crítico

Conversion Rate:
  - % opps que avançaram de stage na última semana
  - Benchmark: >30% = healthy

Stalled Opportunities:
  - Count de opps em "Waiting Feedback" >7 dias
  - Benchmark: <3 = healthy, 3-5 = warning, >5 = crítico

Overdue Actions:
  - Count de opps com Next Action Date no passado
  - Prioridade por Priority field (High → Medium → Low)
```

**Step 3: Identificar Riscos e Oportunidades**
```
🚨 RISCOS:
  - Opps com Days in Stage >14 (ordenar por Priority)
  - Opps "Waiting Feedback" >7 dias sem follow-up
  - Opps High Priority com Next Action Date overdue

💰 OPORTUNIDADES:
  - Opps em "Proposta" ou "Negociação" com next milestone <7 dias
  - Opps que avançaram 2+ stages na última semana (momentum)
```

**Step 4: Gerar Action Items**
```
Para cada risco/oportunidade:
  - Ação recomendada (follow-up, archive, schedule meeting)
  - Contexto mínimo (last Update Log entry, stakeholder)
  - Link direto para a página da Oportunidade
```

### Output (Formato do Digest)

```markdown
# Weekly Pipeline Digest - [Data]

## 📊 Pipeline Health Score: [X/100]

**Velocity:** ⚠️ 12.3 dias (target: <10)
**Conversion:** ✅ 35% avançaram
**Stalled:** 🚨 6 opps >7 dias sem feedback
**Overdue:** ⚠️ 4 High Priority actions atrasadas

---
## 🚨 AÇÃO IMEDIATA NECESSÁRIA (4)

### 1. [Oportunidade X] - 18 dias travada em Proposta
- **Problema:** Waiting Feedback desde 02/11, sem resposta
- **Ação:** Follow-up com [Stakeholder Y] hoje
- **Contexto:** Última interação: "cliente ia revisar proposta internamente"
- **Link:** [Abrir no Notion]

### 2. [Oportunidade Z] - High Priority, Next Action vencida há 3 dias
- **Problema:** "Enviar contrato revisado" era para 17/11
- **Ação:** Enviar contrato hoje ou reagendar
- **Link:** [Abrir no Notion]

[... mais 2 items ...]

---

## 💰 OPORTUNIDADES DE REVENUE (2)

### 1. [Oportunidade A] - Momentum! Avançou Credibilidade → Proposta
- **Próximo Milestone:** Apresentar proposta comercial em 22/11
- **Ação:** Confirmar agenda e preparar briefing (use Agent: Meeting Prep)
- **Link:** [Abrir no Notion]

### 2. [Oportunidade B] - Em Negociação, possível fechamento esta semana
- **Próximo Milestone:** Cliente confirmou aprovação interna
- **Ação:** Preparar contrato e follow-up amanhã
- **Link:** [Abrir no Notion]

---

## 📅 AGENDA DA SEMANA (7 opps com Next Action Date)

**Segunda-feira (hoje):**
- [Opp X]: Follow-up com stakeholder
- [Opp Y]: Enviar contrato revisado

**Terça-feira:**
- [Opp Z]: Reunião de proposta (prep com Agent: Meeting Prep)

**Quarta-feira:**
- [Opp W]: Check-in informal

[... restante da semana ...]

---

## 🔍 PIPELINE OVERVIEW

**Distribuição por Stage:**
- Marketing: 3 opps
- Suspect: 5 opps
- Credibilidade: 4 opps
- Proposta: 6 opps (⚠️ 3 travadas >14 dias)
- Negociação: 2 opps
- Fechamento: 1 opp

**Total Active Pipeline:** 21 opps

---

**Próximo Digest:** Segunda, 27/11 às 9:00 AM
```

### Implementação Técnica

**Comando no My Notion AI:**
```
"Gere o pipeline digest"
ou
"Pipeline digest semanal"
```

**MCP Tools Utilizados:**
1. `mcp__notion__notion-search`: Query oportunidades ativas
2. `mcp__notion__notion-fetch`: Load propriedades de cada opp
3. Processamento local: Cálculo de métricas, ordenação, formatação
4. Output: Markdown estruturado (pode ser enviado por email/Slack futuramente)

**Armazenamento (opcional):**
- Criar página "Pipeline Digests" no Notion
- Cada digest vira subpágina com timestamp
- Permite tracking histórico de métricas semanais

### Métricas de Sucesso

| Métrica | Baseline (Atual) | Target (3 meses) |
|---------|------------------|------------------|
| Tempo de pipeline review | 30 min/semana | 5 min/semana |
| Opps travadas >14 dias | 6 opps | <3 opps |
| Overdue actions não detectadas | ~30% | <5% |
| Taxa de adoção do digest | 0% (não existe) | 90% (usado toda semana) |


