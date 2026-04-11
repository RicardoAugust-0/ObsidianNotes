---
tags: [nexus, edge-function, bff, zero-trust]
created: 2026-04-03
status: 🟢 Produção
category: Sistema
parent: "[[Nexus - Edge Functions]]"
---

# ⚡ process-action

> BFF (Backend for Frontend) com middleware Zero Trust. Executa ações de negócio com validação completa.

| Campo | Valor |
|:---|:---|
| **Arquivo** | `supabase/functions/process-action/index.ts` |
| **Linhas** | 149 |
| **Auth** | JWT + email verified + créditos + tenant ownership |

## Ações e Custos
| Action | Créditos | Implementado |
|:---:|:---:|:---:|
| `create_lead` | 0 | ✅ |
| `update_lead` | 0 | ✅ |
| `create_pre_lead` | 1 | ❌ (case ausente no switch!) |

## Middleware Chain
```
1. Auth (JWT válido) → 401
2. Email verified (email_confirmed_at) → 403
3. Billing check (require_billing_or_credits) → 402
4. Consume credits (consume_credits) → atomic
5. Tenant ownership (user_id = auth user) → 403
```

## ⚠️ Melhorias
- [x] ~~**🔴 BUG**: `create_pre_lead` não no switch~~ — case já existia no código
- [ ] Poucas ações implementadas — deveria cobrir mais operações
- [x] ~~Não loga em `api_logs`~~ — já implementado via `createLogger`

→ [[Nexus - Melhorias e Roadmap]]
