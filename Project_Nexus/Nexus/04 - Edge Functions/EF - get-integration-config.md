---
tags: [nexus, edge-function, config]
created: 2026-04-03
status: 🟢 Produção
category: Sistema
parent: "[[Nexus - Edge Functions]]"
---

# ⚡ get-integration-config

> Busca configuração de uma integração por tenant (`user_id` + `integration_key`).

| Campo | Valor |
|:---|:---|
| **Arquivo** | `supabase/functions/get-integration-config/index.ts` |
| **Linhas** | 69 |
| **Auth** | JWT via `getClaims()` (extrai `sub` = user_id) |
| **Tabela** | `integration_configs` |

## Input / Output
```typescript
{ integration_key: string }
→ { config: object | null, is_active: boolean }
```

## Diferencial
- Usa `getClaims()` em vez de `getUser()` — mais leve
- Usa `maybeSingle()` — não falha se config não existir

→ [[Nexus - Melhorias e Roadmap]]
