---
tags: [nexus, edge-function, ia, vendas]
created: 2026-04-03
status: 🟢 Produção
category: IA & Análise
parent: "[[Nexus - Edge Functions]]"
---

# ⚡ analyze-opportunities

> Identifica TOP 10 leads mais promissores com scoring IA. Enriquece com interações e attachments.

| Campo | Valor |
|:---|:---|
| **Arquivo** | `supabase/functions/analyze-opportunities/index.ts` |
| **Linhas** | 165 |
| **Modelos** | `gpt-4.1-mini-2025-04-14` → fallback `gemini-2.5-flash` (com tool_choice) |
| **Logging** | ✅ `api_logs` (tokens, custo estimado) |

## Input / Output
```typescript
// Request: sem body (busca no banco)
// Response
{ opportunities: [{ lead_id, lead_name, lead_phone, score, reason, next_action, urgency, recommended_solution }] }
```

## Scoring System
| Status | Pontos Base |
|:---:|:---:|
| Call agendada | 80 |
| Qualificado | 70 |
| Negociação | 60 |
| Proposta | 50 |
| Primeiro contato | 30 |

Bônus: Atividade (+20), Engajamento (+10), Fit Comercial (+10)

## Diferencial
- Usa `tool_choice` (function calling) no fallback Gemini — mais estruturado
- Calcula `days_in_pipeline` e `days_since_update` para cada lead
- Verifica se lead tem proposta PDF (`has_proposal`)

## ⚠️ Melhorias
- [ ] **🔴 CRÍTICO**: Busca leads sem filtro `user_id` — viola multi-tenancy
- [ ] Limite de 50 leads pode excluir oportunidades recentes

→ [[Nexus - Melhorias e Roadmap]]
