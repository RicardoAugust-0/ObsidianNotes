---
tags: [nexus, edge-function, ia, vendas]
created: 2026-04-03
status: 🟢 Produção
category: IA & Análise
parent: "[[Nexus - Edge Functions]]"
---

# ⚡ summarize-chat-history

> Resume o histórico de conversas WhatsApp de um lead e gera análise comercial estruturada.

| Campo           | Valor                                                          |
| :-------------- | :------------------------------------------------------------- |
| **Arquivo**     | `supabase/functions/summarize-chat-history/index.ts`           |
| **Linhas**      | 139                                                            |
| **Modelos**     | `gpt-4.1-mini` → fallback `gemini-2.5-flash`                   |
| **Diferencial** | Usa **Function Calling** (tool_choice) para output estruturado |
| **Logging**     | ✅ `api_logs` (tokens, provider)                                |

## Input / Output
```typescript
// Request
{ leadId: string }
// Response
{ analysis: { nextAction, leadProfile, opportunities, painPoints, maturityLevel, objections, approach },
  totalMessages: number, provider: string }
```

## Comportamento Especial
- Se lead não tem mensagens MAS tem segmento → gera análise baseada só no segmento
- Busca chat pelo `phone` do lead na tabela `n8n_chat_histories_duplicate`

## Diferencial Técnico
- Usa `callAIWithFallback()` — helper abstrato para dual-provider
- Function calling via `analysisToolSchema` — schema tipado para output

→ [[Nexus - Melhorias e Roadmap]]
