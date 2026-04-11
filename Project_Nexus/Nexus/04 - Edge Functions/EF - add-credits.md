---
tags:
  - nexus
  - backend
  - edge-functions
  - creditos
  - supabase
created: 2026-04-08
updated: 2026-04-08
refactored: 2026-04-08
parent: "[[Nexus - Edge Functions]]"
---

# EF — `add-credits`

> Adiciona créditos ao saldo do usuário autenticado. Função administrativa/de billing — **não consome créditos** para executar.

---

## Resumo

| Campo | Valor |
|:---|:---|
| **Arquivo** | `supabase/functions/add-credits/index.ts` |
| **Custo** | 0 créditos (função de billing) |
| **Auth** | JWT obrigatório (`requireAuth`) |
| **Método HTTP** | `POST` |
| **Tabela afetada** | `profiles.credits` |

## Contrato de API

**Request body:**
```json
{ "amount": 10 }
```

**Validação:** `amount` deve ser um inteiro positivo (`number`, `> 0`, `Number.isInteger`). Qualquer outro valor retorna `400 Bad Request`.

**Response (sucesso):**
```json
{ "success": true, "newBalance": 25 }
```

---

## Fluxo de Execução

```
1. CORS preflight (OPTIONS → 200)
2. requireAuth(req) → { user, supabase } — userId extraído do JWT, NUNCA do body
3. Validar amount (inteiro positivo)
4. Tentar incremento atômico via RPC add_credits_atomic
   └─ Se RPC não existir → fallback read-then-write (ainda seguro: userId = JWT)
5. Ler saldo atualizado de profiles WHERE id = user.id
6. logger.success({ duration_ms, amount, new_balance })
7. Retornar { success: true, newBalance }
```

---

## Estratégia de Atomicidade

> [!warning] Fallback não é atômico
> A função tenta usar a RPC `add_credits_atomic` para garantir incremento sem race condition. Se a RPC não existir no banco, ela cai em **read-then-write**: lê o saldo atual e soma `amount`. Isso é seguro contra impersonation (userId vem do JWT), mas **não é thread-safe** em cenários de alta concorrência.

| Estratégia | Condição | Thread-safe? |
|:---|:---|:---:|
| `add_credits_atomic` (RPC) | RPC existe no banco | Sim |
| Read-then-write (fallback) | RPC não encontrada | Não |

---

## Segurança

- **SEC-002:** JWT obrigatório — sem token válido, retorna `401`
- `userId` extraído **exclusivamente** do JWT, nunca do request body — imune a privilege escalation
- ~~A função não verifica se o chamador é admin~~ — `requireAdmin()` adicionado em 2026-04-08.

> [!success] Controle de acesso implementado
> `requireAdmin()` foi adicionado após `requireAuth()`. Apenas usuários com `profiles.role = 'admin'` conseguem chamar esta função. Retorna `403 Forbidden` para qualquer outro perfil.

---

## Interação com o Sistema de Créditos

Esta função é a **fonte de entrada** de créditos no sistema. O fluxo completo:

```
add-credits (entrada) → profiles.credits (saldo)
                      ← consume_credits RPC (saída — chamada pelas outras EFs)
```

Veja [[Nexus - Regras de Negócio e Segurança#Credit Economy]] para o ciclo completo.

---

## Notas Relacionadas

- [[Nexus - Edge Functions]] — Índice de todas as functions
- [[Nexus - Regras de Negócio e Segurança]] — Sistema de créditos completo
- [[Nexus - Shared Helpers]] — `requireAuth`, `createLogger`, `badRequestResponse`
