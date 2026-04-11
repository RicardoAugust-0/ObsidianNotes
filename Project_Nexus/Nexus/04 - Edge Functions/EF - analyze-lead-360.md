---
tags: [nexus, edge-function, ia, vendas]
created: 2026-04-03
status: 🟢 Produção
category: IA & Análise
parent: "[[Nexus - Edge Functions]]"
---

# ⚡ analyze-lead-360

> Análise 360° de um lead para vendas B2B. Identifica dores e recomenda soluções do portfólio RP Consultoria.

| Campo | Valor |
|:---|:---|
| **Arquivo** | `supabase/functions/analyze-lead-360/index.ts` |
| **Linhas** | 195 |
| **Auth** | `getApiKeys()` (tenant OpenAI key) |
| **Modelos** | `gpt-4o-mini` → fallback `gemini-2.5-flash` |

## Input / Output
```typescript
// Request
{ lead: LeadObject, messages: ChatMessage[] }

// Response (JSON com 9 campos)
{ nextAction, leadProfile, opportunities, painPoints, maturityLevel,
  objections, approach, recommendedSolution, solutionFit }
```

## Portfólio Hardcoded
1. **Financeiro** — Cobrança e C.R
2. **Nexus** — Vendas, Prospecção e Análise
3. **Atendimento** — Soluções sob medida
4. **Projetos Exclusivos** — Automações avançadas

## ⚠️ Melhorias
- [x] ~~Portfólio hardcoded no prompt~~ — extraído para constante `PORTFOLIO` no topo do arquivo (2026-04-08). Migração para banco/config pendente (SUG-001).
- [x] ~~Não loga em `api_logs`~~ — já implementado via `createLogger`
- [x] ~~Sem consumo de créditos~~ — já implementado via `consumeCredits`

→ [[Nexus - Melhorias e Roadmap]]
