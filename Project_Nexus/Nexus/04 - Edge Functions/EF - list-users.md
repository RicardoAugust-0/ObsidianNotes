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
- [ ] **Sem validação de role admin** — qualquer JWT válido pode listar todos os users
- [ ] Não filtra campos sensíveis (poderia expor dados indevidos)
- [ ] Deveria paginar resultados

→ [[Nexus - Melhorias e Roadmap]]
