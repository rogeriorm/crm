# Roadmap de Implementação

**Última atualização:** 2025-11-20
**Versão:** 2.0 (Repensado - Abordagem Híbrida)

---

## Estratégia: Incremental Validation

Diferente do roadmap original (que priorizava Weekly Pipeline Digest), a nova estratégia foca em:

1. **Validar abordagem híbrida** criando agente espelho do Process 2.2 existente
2. **Aprender padrões** de implementação antes de criar novos agentes
3. **Construir fundação** para orquestração futura

**Princípio:** Cada fase entrega valor imediato + valida tecnicamente a próxima fase.

---

## Fase 0: Foundation - Agent Mirror 2.2 🎯 AGORA

**Objetivo:** Criar versão Claude Code do Opportunity Advancer (Process 2.2)

### Entregas

1. ✅ **Especificação técnica completa** (`docs/2.2-opportunity-advancer.md`)
   - Input/output specification
   - Lógica de processamento detalhada
   - Casos de teste

2. 🚧 **Implementação do agente mirror**
   - Query Oportunidades por nome
   - Load Anotações relacionadas
   - Parse transcripts/notes
   - Analyze e gerar 5 campos
   - Output: Markdown report para validação

3. 🚧 **Validação contra agente Notion**
   - Testar com 3-5 oportunidades reais
   - Comparar output: Notion AI vs Claude Code
   - Documentar diferenças e refinar lógica

4. 🚧 **Documentação**
   - Como executar o agente
   - Casos de uso
   - Troubleshooting

### Decisões Necessárias

- [ ] **Dry-run mode:** Agente só mostra output sem atualizar Notion? (Recomendado: SIM para começar)
- [ ] **Interatividade:** Exibir 5 campos e pedir confirmação antes de update? (Recomendado: SIM)
- [ ] **Logging:** Salvar execution logs em arquivo? (Recomendado: SIM, criar `logs/` folder)

### Critérios de Sucesso

- [ ] Agente processa oportunidade e gera 5 campos consistentes com Notion AI
- [ ] Output é acionável (user pode copiar ou aprovar update)
- [ ] Tempo de execução < 30 segundos
- [ ] Taxa de acerto > 80% (comparado com validação manual)

### Tempo Estimado

**2-3 horas** distribuídas em:
- Especificação: 30 min (FEITO)
- Implementação core: 1-1.5h
- Testes e refinamento: 30-45 min
- Documentação: 15-30 min

---

## Fase 1: First New Agent - Weekly Pipeline Digest

**Objetivo:** Criar primeiro agente net-new (não existe no Notion AI)

### Entregas

1. ✅ **Especificação técnica** (`docs/3.0-weekly-pipeline.md`) - já existe!

2. 🔜 **Implementação do agente**
   - Query oportunidades ativas (via MCP search)
   - Load properties de cada opp (via MCP fetch)
   - Calcular métricas (velocity, stalled, overdue)
   - Gerar digest estruturado

3. 🔜 **Iteração do formato**
   - Testar com dados reais por 2 semanas
   - Ajustar thresholds (o que é "stalled"? >7 ou >14 dias?)
   - Refinar action items (nível de detalhe ideal?)

4. 🔜 **Opcional: Salvar no Notion**
   - Criar página "Pipeline Digests"
   - Cada execução vira subpágina com timestamp
   - Permite tracking histórico de métricas

### Decisões Necessárias

- [ ] **Frequência:** Sob demanda ou scheduled (via cron/GitHub Actions)?
- [ ] **Formato de output:** Markdown no terminal, arquivo, ou página Notion?
- [ ] **Nível de detalhe:** Digest compacto (1 página) ou completo (2-3 páginas)?

### Critérios de Sucesso

- [ ] Digest gerado em < 60 segundos
- [ ] Identifica corretamente opps travadas e overdue
- [ ] Action items são acionáveis (clear next steps)
- [ ] User usa digest ativamente (>80% das semanas)

### Tempo Estimado

**3-4 horas** distribuídas em:
- Implementação core: 1.5-2h
- Testes e refinamento: 1h
- Integração opcional Notion: 30 min
- Documentação: 30 min

**Iteração do formato:** 2 semanas de uso ativo

---

## Fase 2: Expansion - Meeting Prep & Opportunity Creator

**Objetivo:** Adicionar 2 agentes de alto impacto no dia a dia

### 2.1 - Meeting Prep Briefer

**Valor:** Economiza 10-15 min/reunião (50 min/semana se 5 reuniões)

**Entregas:**
- Load histórico da oportunidade (Update Log, últimas Anotações)
- Gerar SPIN questions adaptadas ao Biz Funnel stage
- Sugerir objetivos da reunião
- Output: Briefing 1-2 páginas (markdown)

**Tempo estimado:** 2-3 horas

### 1.0 - Opportunity Creator

**Valor:** Economiza 10 min/oportunidade nova (setup manual)

**Entregas:**
- Template para criação rápida
- Auto-link parent task (se workshop/evento)
- Copy contacts relacionados
- Set initial Update Log
- Output: Oportunidade criada no Notion

**Tempo estimado:** 2-3 horas

### Critérios de Sucesso (Fase 2)

- [ ] Meeting Prep usado em >70% das reuniões importantes
- [ ] Opportunity Creator reduz tempo de setup em >50%
- [ ] Ambos integram smoothly com workflow existente

### Tempo Estimado Total

**4-6 horas** distribuídas em 3-4 semanas

---

## Fase 3: Communication - Follow-up Drafter

**Objetivo:** Automatizar drafting de emails de follow-up

**Valor:** Economiza 15 min/semana + melhora consistência

### Entregas

- Load contexto da última Anotação
- Adapt tone baseado em Biz Funnel stage (Credibilidade = mais educativo, Negociação = mais direto)
- Gerar draft de email personalizado
- Output: Markdown com email pronto para copiar

### Decisões Necessárias

- [ ] **Tom de comunicação:** Formal, semiformal, ou casual?
- [ ] **Nível de personalização:** 80% ready ou 50% skeleton?
- [ ] **Templates:** Criar biblioteca de templates por stage?

### Critérios de Sucesso

- [ ] Drafts precisam de <5 min de edição antes de enviar
- [ ] Taxa de uso >60% (usado em >60% dos follow-ups)
- [ ] User reporta que follow-ups são mais consistentes

### Tempo Estimado

**3-4 horas** distribuídas em 2-3 semanas

---

## Fase 4: Orchestration - Integrated Workflows

**Objetivo:** Coordenar múltiplos agents em workflows end-to-end

**Valor:** Zero esforço manual para workflows completos

### Entregas

1. **Event Detection System**
   - Detect nova Anotação criada (polling ou webhook)
   - Detect Next Action Date vencida
   - Detect Biz Funnel stage change

2. **Workflow Orchestrator**
   - Define workflows (ex: Nova Anotação → Advance → Meeting Prep)
   - Execute agents em sequência
   - Handle erros e fallbacks

3. **Dashboard de Monitoramento**
   - Execuções por agent/semana
   - Tempo economizado (estimado)
   - Taxa de sucesso/erro

### Exemplo de Workflow Orquestrado

**Trigger:** Nova página criada em "Anotações" (reunião com cliente)

**Workflow:**
1. Orchestrator detecta: Anotação linkada à Oportunidade X
2. Chama Agent 2.2 (Opportunity Advancer) → Preenche 5 campos
3. Detecta: Biz Funnel avançou de Credibilidade → Proposta
4. Chama Agent 2.1 (Meeting Prep Briefer) → Gera briefing para próxima reunião
5. Chama Agent 4.0 (Follow-up Drafter) → Gera thank-you email

**Resultado:** Pipeline avançado + próxima reunião preparada + follow-up rascunhado - tudo automático

### Decisões Necessárias

- [ ] **Nível de automação:** Orchestrator executa automaticamente ou sempre pede confirmação?
- [ ] **Error handling:** Retry automático, fallback para manual, ou alert?
- [ ] **Monitoring:** Dashboard web, CLI, ou apenas logs?

### Critérios de Sucesso

- [ ] Pelo menos 1 workflow end-to-end funcionando (ex: Anotação → Advance → Prep)
- [ ] Redução de >70% do esforço manual em workflows orquestrados
- [ ] Sistema confiável (uptime >95%, error rate <5%)

### Tempo Estimado

**8-12 horas** distribuídas em 4-6 semanas

---

## Timeline Visual

```
Fase 0: Agent Mirror 2.2           [===]                      (Semana 1)
         └─> 2-3 horas, valida abordagem

Fase 1: Weekly Pipeline Digest      [====]                    (Semanas 2-3)
         └─> 3-4 horas + 2 semanas iteração

Fase 2: Meeting Prep & Opp Creator      [=======]             (Semanas 4-7)
         └─> 4-6 horas distribuídas

Fase 3: Follow-up Drafter                    [=====]          (Semanas 8-10)
         └─> 3-4 horas distribuídas

Fase 4: Orchestration                            [===========] (Semanas 11-16)
         └─> 8-12 horas distribuídas

─────────────────────────────────────────────────────────────────
0        2        4        6        8       10       12       14       16
                                                            (semanas)
```

---

## Métricas de Progresso

### Tempo Economizado (Acumulado)

| Após Fase | Agentes Ativos | Tempo/Semana Economizado | Acumulado |
|-----------|----------------|--------------------------|-----------|
| Fase 0 | 1 (Mirror 2.2) | ~5 min (validação) | ~5 min |
| Fase 1 | 2 (Digest) | ~5 + 30 min | ~35 min |
| Fase 2 | 4 (Meeting Prep + Creator) | ~35 + 50 + 10 min | ~95 min |
| Fase 3 | 5 (Follow-up) | ~95 + 15 min | ~110 min |
| Fase 4 | 5 (Orquestrados) | ~110 + 30 min (orquestração) | ~140 min |

**Meta Final:** Economizar **140 minutos/semana (2h20min)** com automação

### Taxa de Adoção Target

- **Fase 0:** N/A (validação técnica)
- **Fase 1:** >80% (digest usado toda semana)
- **Fase 2:** >70% (agents usados na maioria dos casos)
- **Fase 3:** >60% (drafts usados em >60% dos follow-ups)
- **Fase 4:** >50% (pelo menos metade dos workflows orquestrados)

---

## Riscos por Fase

| Fase | Risco Principal | Mitigação |
|------|----------------|-----------|
| **Fase 0** | Mirror não replica Notion AI com fidelidade | Testar com casos reais, iterar lógica, aceitar 80% accuracy |
| **Fase 1** | Digest não é acionável (muito genérico) | Iterar formato por 2 semanas com dados reais |
| **Fase 2** | Meeting Prep é muito longo (não economiza tempo) | Testar formatos: brief vs comprehensive, ajustar |
| **Fase 3** | Follow-up drafts precisam de muita edição | Refinar com templates, ajustar tom e personalização |
| **Fase 4** | Orquestração é muito complexa (bugs, falhas) | Começar com 1 workflow simples, adicionar error handling robusto |

---

## Próximos Passos Imediatos

### Esta Semana (Fase 0)

1. ✅ **Especificar Agent Mirror 2.2** → `docs/2.2-opportunity-advancer.md`
2. 🚧 **Implementar lógica core** → Query, load, parse, analyze
3. 🚧 **Testar com 3-5 oportunidades reais**
4. 🚧 **Documentar como executar**

### Semana 2-3 (Fase 1)

5. 🔜 Implementar Weekly Pipeline Digest
6. 🔜 Testar formato por 2 semanas
7. 🔜 Iterar baseado em feedback real

### Decisão Gate: Após Fase 1

**Avaliar:** Abordagem híbrida está funcionando? Agents entregam valor?

**Se SIM:** Prosseguir para Fase 2
**Se NÃO:** Repensar estratégia, possivelmente focar só em Notion AI ou só em Claude Code

---

**Fim do Cronograma**
**Versão:** 2.0
**Próxima Revisão:** Após conclusão da Fase 0
