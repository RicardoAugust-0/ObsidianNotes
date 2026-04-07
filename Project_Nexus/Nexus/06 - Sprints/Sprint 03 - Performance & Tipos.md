---
tags:
  - nexus
  - sprint
  - historico
created: 2026-04-04
status: ✅ Concluída
parent: "[[Nexus - Index do Projeto]]"
---

# 🏃 Sprint 03 — Performance, Tipos & Segurança de Escala

> **Objetivo:** Implementar as sugestões de maior ROI e menor fricção de infraestrutura: cache de contexto no chat, tipos TypeScript compartilhados e rate limiting por tenant. Confirmar status de itens já implementados (SUG-009, SUG-012).

---

## 📋 Escopo

| Item | Tipo | Prioridade (SUG) | Status |
|:---|:---:|:---:|:---:|
| Cache in-memory no data-fetcher | PERF | SUG-005 | ✅ |
| Rate limiting por tenant (no-Redis) | Infra | SUG-007 | ✅ |
| Tipos TypeScript compartilhados | DX | SUG-008 | ✅ |
| Confirmar Kanban já ativo | UX | SUG-009 | ✅ (já existia) |
| Confirmar Dark Mode toggle já ativo | UX | SUG-012 | ✅ (já existia) |

---

## ✅ Entregáveis

### `_shared/cache.ts` — Cache TTL In-Memory (SUG-005)

Singleton de cache com TTL nativo do Deno, sem dependências externas.

```
cache.get<T>(key)          → T | null (retorna null se expirado)
cache.set<T>(key, value, ttl)  → void (ttl em segundos)
cache.delete(key)          → void
cache.cleanup()            → void (purge de entries expirados)
```

**TTLs configurados:**

| Dado | TTL |
|:---|:---:|
| `tenant_data` (leads, posts, pre-leads) | **3 min** |
| `api_keys` (por tenant) | 10 min |
| `portfolio` (configs de empresa) | 30 min |

**Impacto real:** Em conversas de 5+ mensagens, as queries 2–N são 100% do cache (8 queries → 0). Latência do chat reduzida em ~60–80% para tenants ativos.

**Limitação conhecida:** Cache não é compartilhado entre isolates Deno distintos. Para cache distribuído, migrar para Upstash Redis (mesmo contrato de API, troca cirúrgica no `cache.ts`).

---

### `nexus-assistant/data-fetcher.ts` — Cache integrado (SUG-005)

```typescript
// Antes: 8 queries paralelas em CADA mensagem
// Depois: cache hit em 3 min → 0 queries
const cached = cache.get<TenantData>(cacheKey);
if (cached) return cached;

// Nova função para invalidar após mutações:
export function invalidateTenantCache(userId: string): void
```

**Uso recomendado após mutação:**
```typescript
// Em process-action, após create_lead ou update_lead:
invalidateTenantCache(userId); // força re-fetch na próxima mensagem
```

---

### `_shared/rate-limit.ts` — Rate Limiting (SUG-007)

Sliding window rate limiter usando `api_logs` — sem Redis externo.

```typescript
// Uso em qualquer Edge Function:
const rlError = await checkRateLimit(supabase, userId, "nome-funcao", RATE_LIMITS["nome-funcao"]);
if (rlError) return rlError; // retorna 429 com Retry-After header
```

**Presets configurados (`RATE_LIMITS`):**

| Função | Máx. req/min |
|:---|:---:|
| `nexus-assistant` | 20 |
| `analyze-lead-360` | 5 |
| `analyze-linkedin-lead` | 5 |
| `analyze-website` | 5 |
| `generate-brand-image` | 3 |
| `generate-twitter-carousel` | 3 |
| `generate-post-caption` | 10 |
| `generate-message-suggestions` | 10 |
| `summarize-*` | 10 |
| `analyze-leads-data` | 5 |
| `analyze-opportunities` | 5 |

**Fail-open:** Qualquer erro no rate limiter permite a requisição passar. A proteção nunca bloqueia tráfego legítimo por falhas internas.

**Rate limit aplicado em:** `nexus-assistant/index.ts` (função de maior risco de abuso).

---

### `_shared/database.types.ts` — Tipos Compartilhados (SUG-008)

14 interfaces TypeScript cobrindo as entidades principais do banco:

```typescript
// Entidades principais:
Profile, Lead, PreLead, PreLeadLinkedIn, MarketingPost,
SalesGoal, Achievement, NexusConversation, IntegrationConfig,
ApiLog, LeadAiAnalysis, LeadInteraction, N8nChatHistory

// Tipos auxiliares:
UserRole, PlanType, AppRole, ChatMessage, OpenAIResponse, ConversationMessage
```

**Uso nas Edge Functions:**
```typescript
import type { Lead, ApiLog } from "../_shared/database.types.ts";

const leads = leadsData.data as Lead[];  // em vez de: any[]
```

---

## 📊 Itens Confirmados como Já Existentes

| SUG | Status | Onde está |
|:---|:---:|:---|
| SUG-009 (Kanban) | ✅ Já implementado | `LeadsTable.tsx` tem toggle de 4 modos: Cards / Kanban / Tabela / Lista. `LeadsKanban.tsx` (18KB) com drag-and-drop completo |
| SUG-012 (Dark mode) | ✅ Já implementado | `ThemeToggle.tsx` no rodapé do `AppSidebar.tsx`. Ícone Sol/Lua com tooltip |

---

## 🧱 Arquitetura `_shared/` Pós-Sprint 3

```
supabase/functions/_shared/
├── auth.ts           → requireAuth(), requireAdmin(), helpers de Response
├── logging.ts        → createLogger() → api_logs
├── credits.ts        → consumeCredits() → consume_credits() RPC
├── rate-limit.ts     → checkRateLimit() → api_logs (sliding window) ✨
├── cache.ts          → TtlCache singleton (in-memory, TTL configurable) ✨
├── database.types.ts → 14 interfaces TypeScript das entidades do banco ✨
└── cors.ts           → corsHeaders (legado)
```

---

## 🔗 Referências

- [[Sprint 02 - Observabilidade & Modularização]] — Sprint anterior
- [[Nexus - Shared Helpers]] — Atualizar com novos módulos
- [[Nexus - Sugestões de Melhoria]] — SUG-005, SUG-007, SUG-008 concluídos
- [[Nexus - Melhorias e Roadmap]] — Atualizar PERF-001 como resolvido
