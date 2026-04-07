---
tags: [nexus, edge-function, ia, analytics]
created: 2026-04-03
status: 🟢 Produção
category: IA & Análise
parent: "[[Nexus - Edge Functions]]"
---

# ⚡ analyze-leads-data

> Análise NLP em batch de todos os leads. Extrai dores, palavras-chave e insights de ML.

| Campo | Valor |
|:---|:---|
| **Arquivo** | `supabase/functions/analyze-leads-data/index.ts` |
| **Linhas** | 168 |
| **Modelos** | `gpt-4o-mini` → fallback `gemini-2.5-flash` |
| **Dados** | Todos os leads (sem filtro de tenant!) |

## Input / Output
```typescript
// Request: sem body (busca todos os leads)
// Response
{ painPoints: [{category, items[], count}], topWords: [{word, count}], mlInsights: [{type, title, description, impact}] }
```

## ⚠️ Melhorias
- [ ] **🔴 CRÍTICO**: Busca TODOS os leads sem filtro de `user_id` — viola multi-tenancy
- [ ] Sem limite de registros (`.limit()` ausente)
- [ ] Trunca informações em 8000 chars — pode perder dados
- [ ] Não loga em `api_logs`

→ [[Nexus - Melhorias e Roadmap]]
