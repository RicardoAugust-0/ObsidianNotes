---
tags:
  - nexus
  - sprint
  - historico
  - seguranca
created: 2026-04-05
status: ✅ Concluída
parent: "[[Nexus - Index do Projeto]]"
---

# 🏃 Sprint 05 — Revisão de Segurança Backend

> **Objetivo:** Revisão completa de todas as 20+ Edge Functions e módulos `_shared/`. Aplicar correções para vulnerabilidades críticas (SSRF, prompt injection, cross-tenant data leak, column injection, query injection) e resolver gaps de segurança identificados.

---

## 📋 Escopo

| Item | Severidade | Descrição |
|:---|:---|:---|
| SEC-001 | 🔴 | Nexus-assistant permitia acesso anônimo — vazamento de dados cross-tenant |
| SEC-002 | 🔴 | `data-fetcher.ts` executava queries sem filtro `user_id` quando userId=null |
| SEC-003 | 🔴 | `generate-chat-embeddings` sem consumo de crédito + Lovable key enviada para endpoint OpenAI |
| SEC-004 | 🔴 | Prompt injection via dados editáveis pelo usuário no `prompt-builder.ts` |
| SEC-005 | 🔴 | SSRF em `analyze-website` e `generate-twitter-carousel` — URL do usuário fetchada sem validação |
| SEC-006 | 🔴 | `process-action` inseria dados do corpo sem whitelist — column injection |
| SEC-007 | 🔴 | Query injection no Serper.dev em `search-leads` — parâmetros sem escape |
| SEC-008 | 🟡 | `summarize-meeting-prep` query em `reunioes_sdr` sem filtro `user_id` |
| SEC-009 | 🟡 | Import path inválido em `flows/linkedin.ts` (`./types.ts` → `../types.ts`) |
| SEC-010 | 🟡 | Mensagem de erro interno vazada ao cliente em `_shared/credits.ts` |
| SEC-011 | 🟡 | `supabase: any` em `_shared/rate-limit.ts` — sem type safety |
| SEC-012 | 🟡 | Payload de log sem sanitização — risco de vazar dados sensíveis |
| SEC-013 | 🟢 | Error messages internas expostas em múltiplos catch handlers |
| SEC-014 | 🟢 | Sem entrada de crédito para `generate-chat-embeddings` e `search-leads` |
| SEC-015 | 🟢 | Validação fraca de campos AI em `analyze-lead-360` (usava `!` truthiness) |
| SEC-016 | 🟢 | `JSON.parse` sem try/catch em `flows/email.ts` |
| SEC-017 | 🟢 | Error leak no catch de `search-leads` |
| SEC-018 | 🟢 | Fetch sem timeout em `analyze-website` — risco de hanging request |

---

## ✅ Entregáveis

### Correções Críticas (7 fixes)

| Arquivo | Mudança |
|:---|:---|
| `nexus-assistant/index.ts` | `requireAuth` obrigatório — removido try/catch que permitia acesso anônimo. Credit check sem null guard. |
| `nexus-assistant/data-fetcher.ts` | Removido fallback "sem filtro" — todas as 8 queries agora têm `.eq("user_id", userId)` obrigatório. |
| `generate-chat-embeddings/index.ts` | Adicionado `consumeCredits` (5 créditos). Só usa OpenAI key para endpoint OpenAI. |
| `nexus-assistant/prompt-builder.ts` | Tags XML (`<lead_card>`, `<opportunity_card>`, etc.) + `escapeForPrompt()` com trunc 500 chars + instrução anti-injection no system prompt. |
| `analyze-website/index.ts` | `isSafePublicUrl()` bloqueia IPs privados e metadata endpoints. Timeout 15s com AbortController. |
| `generate-twitter-carousel/index.ts` | Mesmo `isSafePublicUrl()` + timeout na fetch de URL. |
| `process-action/index.ts` | Whitelist explícita de campos permitidos para `create_lead` e `update_lead`. |
| `search-leads/index.ts` | `escapeSerper()` com escape de caracteres especiais da query. |

### Correções Importantes (5 fixes)

| Arquivo | Mudança |
|:---|:---|
| `summarize-meeting-prep/index.ts` | Adicionado `.eq("user_id", user.id)` na query de `reunioes_sdr`. |
| `flows/linkedin.ts` | Import corrigido para `"../types.ts"`. JSON.parse com try/catch. |
| `flows/email.ts` | JSON.parse com try/catch para LLM retornando JSON malformado. |
| `_shared/credits.ts` | Mensagem de erro genérica. Log com user ID truncado (8 chars). |
| `_shared/rate-limit.ts` | Tipado com `SupabaseClient` em vez de `any`. |
| `_shared/logging.ts` | Sanitização de metadata: strip sensitive keys (api_key, token, secret, password, authorization, service_role), truncate 1024 chars. |

### Correções Menores (6 fixes)

| Arquivo | Mudança |
|:---|:---|
| `analyze-lead-360/index.ts` | Type check `typeof analysis[field] !== "string"` em vez de `!analysis[field]`. `content!` → `content?.`. Error message genérica. |
| `search-leads/index.ts` | Error message genérica no catch principal. |
| `_shared/credits.ts` | Adicionados `generate-chat-embeddings` (5) e `search-leads` (1) ao mapa de custos. |

---

## 📊 Impacto

- **0 → 18 correções aplicadas** em 15 arquivos
- **Cross-tenant data leak:** Resolvido (SEC-001 + SEC-002)
- **SSRF:** Resolvido (SEC-005) — URLs privadas bloqueadas
- **Prompt injection:** Mitigado (SEC-004) — dados envoltos em tags XML + sanitização
- **Column injection:** Resolvido (SEC-006) — whitelists explícitas
- **Query injection:** Resolvido (SEC-007) — escape de caracteres especiais
- **Credit bypass:** Resolvido (SEC-003, SEC-014) — consumo de crédito em todas as funções

## ⚠️ Deploy Required

> Todas as Edge Functions alteradas **exigem deploy** via Supabase Dashboard antes de entrarem em produção.

---

## 🔗 Referências

- [[Nexus - Melhorias e Roadmap]] — Status completo de todos os itens
- [[Nexus - Shared Helpers]] — Detalhes dos módulos atualizados
- [[EF - nexus-assistant]] — Documentação atualizada com mudanças de segurança
