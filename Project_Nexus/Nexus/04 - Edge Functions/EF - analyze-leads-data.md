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
- [x] ~~**🔴 CRÍTICO**: Busca TODOS os leads sem filtro de `user_id`~~ — filtro `.eq("user_id", user.id)` já existia
- [x] ~~Sem limite de registros~~ — `.limit(500)` adicionado (2026-04-08)
- [x] ~~Trunca informações em 8000 chars — pode perder dados~~ — truncação por lead (500 chars cada) implementada (2026-04-08)
- [x] ~~Não loga em `api_logs`~~ — já implementado via `createLogger`

→ [[Nexus - Melhorias e Roadmap]]
