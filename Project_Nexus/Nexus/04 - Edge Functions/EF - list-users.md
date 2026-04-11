---
tags: [nexus, edge-function, admin]
created: 2026-04-03
status: 🟢 Produção
category: Sistema
parent: "[[Nexus - Edge Functions]]"
---

# ⚡ list-users

> Lista todos os usuários do Supabase Auth. Acesso admin-only.

| Campo | Valor |
|:---|:---|
| **Arquivo** | `supabase/functions/list-users/index.ts` |
| **Linhas** | 72 |
| **Auth** | JWT + Service Role (`auth.admin.listUsers()`) |

## Input / Output
```typescript
// Request: sem body
→ [{ id: string, email: string }]
```

## ⚠️ Melhorias
- [x] ~~**Sem validação de role admin**~~ — `requireAdmin()` já existia
- [x] ~~Não filtra campos sensíveis~~ — retorna apenas `{ id, email }`
- [x] ~~Deveria paginar resultados~~ — paginação via `?page=&perPage=` implementada (2026-04-08)

→ [[Nexus - Melhorias e Roadmap]]
