---
tags: [nexus, edge-function, ia, vendas]
created: 2026-04-03
status: 🟢 Produção
category: IA & Análise
parent: "[[Nexus - Edge Functions]]"
---

# ⚡ generate-message-suggestions

> Gera sugestões de mensagens WhatsApp personalizadas para abordagem de leads.

| Campo | Valor |
|:---|:---|
| **Arquivo** | `supabase/functions/generate-message-suggestions/index.ts` |
| **Linhas** | 143 |
| **Modelos** | `gpt-4o-mini` → fallback `gemini-2.5-flash` |

## Input / Output
```typescript
// Request
{ lead: LeadObject, messages: ChatMessage[] }
// Response
{ analysis, identified_pain, recommended_solution, connection_point,
  suggestions: [{ type, label, message, reason }] }
```

## Tipos de Sugestão
- `engajamento` — Retomar contato
- `qualificacao` — Entender necessidade
- `agendamento` — Agendar call
- `fechamento` — Fechar negócio

## Diferencial
- Usa últimas 30 mensagens como contexto
- Regra de máximo 200 chars por sugestão
- Fallback local com mensagens genéricas se IA falhar

## ⚠️ Melhorias
- [ ] Não loga em `api_logs`
- [ ] Sugestões genéricas no fallback local perdem contexto

→ [[Nexus - Melhorias e Roadmap]]
