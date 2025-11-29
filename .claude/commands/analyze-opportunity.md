Apply the opportunity-advancer agent to analyze the opportunity named "{{ARG}}".

Follow the complete SPAR workflow defined in `.claude/agents/opportunity-advancer.md`:
- SENSE: Search, validate, fetch opportunity + load interactions
- PLAN: Multi-source strategic analysis
- ACT: Generate 6 field recommendations
- REFLECT: Log to memory with execution_context: "slash_command"

**Agent spec:** `.claude/agents/opportunity-advancer.md`
**Opportunity name:** {{ARG}}
