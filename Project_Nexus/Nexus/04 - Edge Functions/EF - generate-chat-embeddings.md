---
tags: [nexus, edge-function, embeddings, busca-semantica]
created: 2026-04-03
status: 🟢 Produção
category: Conteúdo & Mídia
parent: "[[Nexus - Edge Functions]]"
---

# ⚡ generate-chat-embeddings

> Gera embeddings vetoriais de conversas WhatsApp e salva na tabela `documents` para busca semântica.

| Campo | Valor |
|:---|:---|
| **Arquivo** | `supabase/functions/generate-chat-embeddings/index.ts` |
| **Linhas** | 91 |
| **Modelo** | `text-embedding-ada-002` (OpenAI) |
| **Tabelas** | `n8n_chat_histories_duplicate` → `documents` |

## Input / Output
```typescript
// Request: sem body
// Response
{ success: boolean, processed: number, total: number }
```

## Processo
1. Busca últimas 100 conversas de `n8n_chat_histories_duplicate`
2. Para cada conversa, monta string: `Lead ID + Session + Mensagem`
3. Gera embedding via `text-embedding-ada-002`
4. Insere em `documents` com metadata (lead_id, session_id)
5. Batch de 10 com delay de 100ms entre batches

## ⚠️ Melhorias
- [x] ~~**Sem de-duplicação**~~ — de-duplicação via `alreadyEmbedded` já existia
- [x] ~~Limite de 100 registros — não processa histórico completo~~ — aumentado para `.limit(500)` (2026-04-08)
- [x] ~~Sem filtro de tenant (`user_id`)~~ — filtro por `tenantLeadIds` já existia
- [ ] Deveria ser executada como cron/trigger, não on-demand

→ [[Nexus - Melhorias e Roadmap]]
