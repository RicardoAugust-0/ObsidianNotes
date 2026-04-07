---
tags:
  - nexus
  - roadmap
  - melhorias
  - bugs
  - backlog
created: 2026-04-03
updated: 2026-04-05
parent: "[[Nexus - Index do Projeto]]"
---

# 🚀 Nexus — Melhorias e Roadmap

> Lista completa de melhorias, bugs e novas funcionalidades identificadas pela auditoria de código do [[Nexus - Index do Projeto|Nexus]]. Organizado por **prioridade** e **área**.

## 📦 Sprint de Revisão de Segurança Backend — `2026-04-05` ✅ CONCLUÍDA

> Revisão completa de todas as 20+ Edge Functions + módulos compartilhados. 18 correções aplicadas.

### Correções Aplicadas

**Críticas (7 fix):**

| # | Arquivo | Problema | Fix |
|:---|:---|:---|:---|
| SEC-001 | `nexus-assistant/index.ts` | Acesso anônimo permitia vazamento cross-tenant | `requireAuth` agora obrigatório (try/catch removido) |
| SEC-002 | `nexus-assistant/data-fetcher.ts` | Queries sem filtro `user_id` quando userId=null | Todas queries agora com `.eq("user_id", userId)` obrigatório |
| SEC-003 | `generate-chat-embeddings/index.ts` | Sem consumo de crédito + Lovable key enviada para endpoint OpenAI | `consumeCredits` adicionado (5 créditos) + só usa OpenAI key |
| SEC-004 | `nexus-assistant/prompt-builder.ts` | Prompt injection via dados editáveis pelo usuário | Tags XML + `escapeForPrompt()` com trunc 500 chars + instrução anti-injection |
| SEC-005 | `analyze-website/index.ts`, `generate-twitter-carousel/index.ts` | SSRF — URL do usuário fetchada sem validação | `isSafePublicUrl()` bloqueia IPs privados/metadata endpoints + timeout 15s |
| SEC-006 | `process-action/index.ts` | Dados do body inseridos sem whitelist | Whitelist explícita de campos para create_lead e update_lead |
| SEC-007 | `search-leads/index.ts` | Query injection no Serper.dev | `escapeSerper()` com escape de chars especiais |

**Importantes (5 fix):**

| # | Arquivo | Problema | Fix |
|:---|:---|:---|:---|
| SEC-008 | `summarize-meeting-prep/index.ts` | Query reunioes_sdr sem filtro tenant | Adicionado `.eq("user_id", user.id)` |
| SEC-009 | `flows/linkedin.ts` | Import path `./types.ts` inválido | Corrigido para `"../types.ts"` |
| SEC-010 | `_shared/credits.ts` | Mensagem de erro interno vazada ao frontend | Mensagem genérica sempre |
| SEC-011 | `_shared/rate-limit.ts` | `supabase: any` sem type safety | Importado `SupabaseClient` tipado |
| SEC-012 | `_shared/logging.ts` | Metadata sem sanitização — risco de vazar secrets | Sanitização: strip sensitive keys, truncate 1024 chars |

**Menores (6 fix):**

| # | Arquivo | Problema | Fix |
|:---|:---|:---|:---|
| SEC-013 | Vários | Error messages internas no catch principal | Mensagens genéricas |
| SEC-014 | `_shared/credits.ts` | Sem entrada para `generate-chat-embeddings` e `search-leads` | Adicionados (5 e 1 crédito) |
| SEC-015 | `analyze-lead-360/index.ts` | Validação fraca (`!` truthiness em campos AI) | Type check `typeof analysis[field] !== "string"` |
| SEC-016 | `flows/email.ts` | `JSON.parse` sem try/catch | Wrapped em try/catch |
| SEC-017 | `search-leads/index.ts` | Error leak no catch | Mensagem genérica |
| SEC-018 | `analyze-website/index.ts` | Fetch sem timeout | AbortController 15s |

**Arquivos Modificados:**
- `supabase/functions/nexus-assistant/index.ts` — Auth obrigatória
- `supabase/functions/nexus-assistant/data-fetcher.ts` — Tenant isolation obrigatório
- `supabase/functions/nexus-assistant/prompt-builder.ts` — Prompt injection prevention
- `supabase/functions/nexus-assistant/flows/linkedin.ts` — Import fix + JSON.parse seguro
- `supabase/functions/nexus-assistant/flows/email.ts` — JSON.parse seguro
- `supabase/functions/analyze-website/index.ts` — SSRF prevention + timeout
- `supabase/functions/generate-twitter-carousel/index.ts` — SSRF prevention
- `supabase/functions/generate-chat-embeddings/index.ts` — Credit consumption + key routing
- `supabase/functions/process-action/index.ts` — Whitelist de campos
- `supabase/functions/search-leads/index.ts` — Query injection prevention + error leak fix
- `supabase/functions/summarize-meeting-prep/index.ts` — Tenant isolation
- `supabase/functions/_shared/credits.ts` — Credit costs + error message sanitization + log truncation
- `supabase/functions/_shared/logging.ts` — Payload sanitization
- `supabase/functions/_shared/rate-limit.ts` — Type safety
- `supabase/functions/analyze-lead-360/index.ts` — Type validation + error leak fix

### ⚠️ Deploy Required

> Estas correções **exigem deploy** das Edge Functions alteradas via painel do Supabase Dashboard antes de entrarem em produção.

---

## 📦 Sprint de Melhorias de Qualidade Backend — `2026-04-05` ✅ CONCLUÍDA

> Revisão adicional pós-deploy: **12 funções** com error leak corrigido (mensagens genéricas no catch), validação de template name no delete do whatsapp-templates, optional chaining fix em `analyze-leads-data`, e remoção de info leak em `summarize-meeting-prep`.

Para detalhes completos, veja: [[Sprint 06 - Melhorias de Qualidade Backend]]

---

## 📦 Sprint de Infra & Segurança — `2026-04-04` ✅ CONCLUÍDA

> Esta sprint resolveu todos os itens **Críticos** (🔴) e os principais itens de **Alto Impacto** (🟠) identificados na auditoria inicial.

### O que foi implementado

**Fase 1 — Shared Helpers (novos arquivos em `_shared/`)**

| Arquivo | Propósito |
|:---|:---|
| `_shared/auth.ts` | `requireAuth()` para validar JWT + `requireAdmin()` para verificar role · resolve SEC-001 / SEC-002 |
| `_shared/logging.ts` | `logApiCall()` padronizado + `createLogger()` factory · resolve LOG-001 |
| `_shared/credits.ts` | `consumeCredits()` com tabela de custos por função · resolve COST-001 |

**Fase 2 — Funções corrigidas**

| Função | Correções Aplicadas |
|:---|:---|
| [[EF - nexus-assistant]] | SEC-001 · SEC-002 · COST-001 · PERF-001 |
| [[EF - analyze-leads-data]] | SEC-001 · SEC-002 · COST-001 · LOG-001 |
| [[EF - analyze-opportunities]] | SEC-001 · SEC-002 · COST-001 · LOG-001 |
| [[EF - generate-chat-embeddings]] | SEC-001 · SEC-002 · PERF-002 · LOG-001 |
| [[EF - check-achievements]] | SEC-002 (userId extraído do JWT, não do body) · LOG-001 |
| [[EF - list-users]] | SEC-002 (verificação de role `admin`) · LOG-001 |
| [[EF - process-action]] | BUG-001 (`case "create_pre_lead"` implementado) |
| [[EF - test-integration]] | BUG-002 (Meta API `v18.0` → `v23.0`) |

---

## 📦 Sprint de Observabilidade & Modularização — `2026-04-05` ✅ CONCLUÍDA

> Esta sprint completou os itens de **Alto Impacto** restantes (COST-001, LOG-001) em todas as 9 funções IA pendentes, adicionou fallback dual-provider onde faltava (QUAL-002), e modularizou o `nexus-assistant`.

### O que foi implementado

**COST-001 + LOG-001 + SEC-002** — aplicados em todas as funções pendentes:

| Função | COST-001 (créditos) | LOG-001 (logging) | SEC-002 (JWT) | Extras |
|:---|:---:|:---:|:---:|:---|
| [[EF - analyze-lead-360]] | ✅ 2 créditos | ✅ | ✅ | API keys scoped a user_id |
| [[EF - analyze-linkedin-lead]] | ✅ 1 crédito | ✅ | ✅ | API keys scoped a user_id |
| [[EF - analyze-website]] | ✅ 1 crédito | ✅ | ✅ | QUAL-002: fallback Lovable adicionado |
| [[EF - generate-message-suggestions]] | ✅ 1 crédito | ✅ | ✅ | API keys scoped a user_id |
| [[EF - generate-post-caption]] | ✅ 1 crédito | ✅ | ✅ | QUAL-002: fallback Lovable adicionado |
| [[EF - generate-brand-image]] | ✅ 3 créditos | ✅ | ✅ | — |
| [[EF - generate-twitter-carousel]] | ✅ 5 créditos | ✅ | ✅ | — |
| [[EF - summarize-chat-history]] | ✅ 1 crédito | ✅ | ✅ | SEC-001: lead scoped a user_id |
| [[EF - summarize-meeting-prep]] | ✅ 1 crédito | ✅ | ✅ | SEC-001: lead scoped a user_id |
| [[EF - process-action]] | — (sem IA) | ✅ | já tinha | — |

**MOD-001 — Modularização do `nexus-assistant`** (782 linhas → 160 linhas no orquestrador):

| Módulo | Responsabilidade |
|:---|:---|
| `nexus-assistant/index.ts` | Orquestrador slim: roteamento de flows + HTTP response |
| `nexus-assistant/types.ts` | Interfaces TypeScript compartilhadas |
| `nexus-assistant/api-keys.ts` | Busca API keys por tenant em `integration_configs` |
| `nexus-assistant/data-fetcher.ts` | Fetch paralelo das 8 tabelas com filtro de tenant |
| `nexus-assistant/prompt-builder.ts` | Construção do system prompt + resolução de comandos |
| `nexus-assistant/flows/linkedin.ts` | LinkedIn Scout: detecção, coleta, extração de parâmetros |
| `nexus-assistant/flows/email.ts` | Email Search: detecção, coleta, extração de parâmetros |
| `nexus-assistant/flows/lead-lookup.ts` | "Consultar lead:" com query enriquecida |

---

---

## 🔴 Crítico (Segurança / Bugs)

### SEC-001: Funções sem filtro de tenant (user_id)
**Impacto:** 🔴 Dados de todos os tenants expostos
**Funções afetadas:**
- [[EF - analyze-leads-data]] — busca todos os leads sem `.eq('user_id', ...)`
- [[EF - analyze-opportunities]] — mesma falha
- [[EF - generate-chat-embeddings]] — embeda conversas de todos os tenants
- [[EF - nexus-assistant]] — carrega dados de todas as tabelas sem filtro

**Solução:** Adicionar `user_id` como filtro em todas as queries. Extrair do JWT via `requireAuth()`.

**Status:** ✅ **Resolvido em `2026-04-04`**
- `.eq('user_id', userId)` adicionado em **todas** as queries das 4 funções afetadas
- `nexus-assistant`: filtro aplicado em 8 tabelas paralelas (`leads`, `n8n_chat_histories_duplicate`, `marketing_posts`, `pre_lead_linkedin`, `pre_leads`, `sales_goals` + oportunidades)
- `analyze-opportunities`: `lead_interactions` e `lead_attachments` agora filtrados pelos `lead_ids` do tenant (não diretamente por `user_id`)
- `generate-chat-embeddings`: conversas filtradas pelos `lead_ids` do tenant via sub-query

---

### SEC-002: Funções sem autenticação
**Impacto:** 🔴 Qualquer pessoa pode chamar com userId arbitrário
**Funções afetadas:**
- [[EF - check-achievements]] — recebe `userId` no body sem validar JWT
- [[EF - list-users]] — valida JWT mas **não verifica se é admin**

**Solução:** Adicionar JWT + verificação de role admin.

**Status:** ✅ **Resolvido em `2026-04-04`**
- `check-achievements`: `userId` agora extraído exclusivamente do JWT via `requireAuth()`. Parâmetro do body ignorado.
- `list-users`: `requireAdmin()` adicionado — consulta `profiles.role` e retorna 403 se não for `admin`
- Todas as funções IA agora usam `requireAuth()` do helper compartilhado `_shared/auth.ts`

---

### BUG-001: `create_pre_lead` não implementado no switch
**Impacto:** 🟡 Ação declarada no mapa de custos mas cai no `default: throw`
**Função:** [[EF - process-action]]
**Solução:** Adicionar `case "create_pre_lead"` com lógica de inserção em `pre_leads`.

**Status:** ✅ **Resolvido em `2026-04-04`**
- `case "create_pre_lead"` implementado com inserção em `pre_leads`
- Campos injetados automaticamente: `user_id: user.id`, `status: "novo"`, `criado_em: new Date().toISOString()`
- Tenant ownership garantido

---

### BUG-002: Inconsistência de versão da Meta API
**Impacto:** 🟡 Pode causar comportamento inesperado
- [[EF - whatsapp-templates]] usa `v23.0` ✅
- [[EF - test-integration]] testa WhatsApp/Meta com `v18.0` ❌

**Solução:** Unificar para v23.0 em ambas as funções.

**Status:** ✅ **Resolvido em `2026-04-04`**
- `testMeta()`: `v18.0` → `v23.0`
- `testWhatsApp()`: fallback `base_url` atualizado para `v23.0`

---

## 🟠 Alto Impacto (Performance / Custos)

### PERF-001: nexus-assistant carrega milhares de registros por request
**Impacto:** Custo alto de LLM (prompt pode ter 50K+ tokens)
**Função:** [[EF - nexus-assistant]]
**Detalhes:**
- 300 leads + 1000 chats + 200 LinkedIn + 200 Google Maps = ~10K registros
- Tudo serializado como texto no system prompt
- Cada request pode custar $0.10+ em tokens

**Solução:**
1. Filtrar por `user_id` (resolve SEC-001 simultaneamente)
2. Limitar dados no prompt (top 10 leads, top 5 chats recentes)
3. Implementar cache no Redis/KV store com TTL de 5 min
4. Usar summarization prévia em vez de dados brutos

**Status:** ⚠️ **Parcialmente resolvido em `2026-04-04`**
- ✅ Filtro por `user_id` aplicado (resolve cross-tenant)
- ✅ Limites drasticamente reduzidos: leads 300→100, chats 1000→100, LinkedIn 200→50, Google Maps 200→50, oportunidades 100→20, metas 50→20
- ⏳ Cache Redis/KV — **pendente** (próxima sprint)
- ⏳ Summarization prévia — **pendente** (próxima sprint)

---

### PERF-002: generate-chat-embeddings sem de-duplicação
**Impacto:** Reprocessa conversas já embedadas, custos desnecessários
**Função:** [[EF - generate-chat-embeddings]]
**Solução:** Checar se `conversation_id` já existe em `documents.metadata` antes de embedar.

**Status:** ✅ **Resolvido em `2026-04-04`**
- Busca prévia em `documents` para coletar `conversation_id`s já processados
- Conversas já embedadas são ignoradas (skip com log)
- Response agora inclui campo `skipped` com contagem de duplicatas evitadas

---

### COST-001: Falta de consumo de créditos em funções IA
**Impacto:** Usuários free podem usar IA ilimitadamente
**Funções SEM consumo de créditos:**

| Função | Usa IA | Consome Créditos |
|:---|:---:|:---:|
| nexus-assistant | ✅ | ✅ `2026-04-04` |
| analyze-lead-360 | ✅ | ❌ |
| analyze-leads-data | ✅ | ✅ `2026-04-04` |
| analyze-linkedin-lead | ✅ | ❌ |
| analyze-opportunities | ✅ | ✅ `2026-04-04` |
| analyze-website | ✅ | ❌ |
| generate-message-suggestions | ✅ | ❌ |
| summarize-chat-history | ✅ | ❌ |
| summarize-meeting-prep | ✅ | ❌ |
| generate-post-caption | ✅ | ❌ |
| generate-brand-image | ✅ | ❌ |
| generate-twitter-carousel | ✅ | ❌ |

**Solução:** Implementar `consumeCredits()` via `_shared/credits.ts` em todas as funções IA.

**Tabela de custos implementada (em `_shared/credits.ts`):**

| Ação | Créditos | Status |
|:---|:---:|:---:|
| Chat (nexus-assistant) | 1 | ✅ |
| Análise de oportunidades | 2 | ✅ |
| Análise batch de leads | 3 | ✅ |
| Análise 360° | 2 | ✅ `2026-04-05` |
| Sugestão de mensagens | 1 | ✅ `2026-04-05` |
| Summarize chat history | 1 | ✅ `2026-04-05` |
| Summarize meeting prep | 1 | ✅ `2026-04-05` |
| Análise de website | 1 | ✅ `2026-04-05` |
| Análise de perfil LinkedIn | 1 | ✅ `2026-04-05` |
| Geração de legenda (post) | 1 | ✅ `2026-04-05` |
| Geração de imagem | 3 | ✅ `2026-04-05` |
| Geração de carrossel (3 slides) | 5 | ✅ `2026-04-05` |
| Geração de embeddings de chat | 5 | ✅ `2026-04-05` |
| Busca de leads (Serper) | 1 | ✅ `2026-04-05` |

**Status:** ✅ **Completamente resolvido em `2026-04-05`** — todas as 14 funções IA+Search agora consomem créditos.

---

### LOG-001: Falta de logging padronizado
**Impacto:** Impossível monitorar custos e performance
**Funções SEM logging em `api_logs`:**
- analyze-lead-360, analyze-leads-data, analyze-linkedin-lead
- analyze-website, generate-message-suggestions
- generate-post-caption, generate-brand-image, generate-twitter-carousel
- generate-chat-embeddings, process-action
- summarize-meeting-prep, check-achievements, list-users

**Funções COM logging:** ✅
- nexus-assistant, analyze-opportunities, summarize-chat-history

**Solução:** Criar helper compartilhado `logApiCall()` em `_shared/logging.ts`.

**Status:** ✅ **Completamente resolvido em `2026-04-05`**

Todas as funções agora usam `createLogger()` do shared helper:

| Função | Logging adicionado |
|:---|:---:|
| check-achievements | ✅ |
| list-users | ✅ |
| analyze-leads-data | ✅ |
| analyze-opportunities | ✅ |
| generate-chat-embeddings | ✅ |
| nexus-assistant | ✅ |
| analyze-lead-360 | ✅ `2026-04-05` |
| analyze-linkedin-lead | ✅ `2026-04-05` |
| analyze-website | ✅ `2026-04-05` |
| generate-message-suggestions | ✅ `2026-04-05` |
| generate-post-caption | ✅ `2026-04-05` |
| generate-brand-image | ✅ `2026-04-05` |
| generate-twitter-carousel | ✅ `2026-04-05` |
| summarize-chat-history | ✅ `2026-04-05` |
| summarize-meeting-prep | ✅ `2026-04-05` |
| process-action | ✅ `2026-04-05` |


---

## 🟡 Médio Impacto (Qualidade)

### QUAL-001: Portfólio hardcoded nos prompts
**Impacto:** Qualquer mudança no portfólio exige deploy de 5+ funções
**Funções afetadas:** analyze-lead-360, analyze-linkedin-lead, analyze-opportunities, analyze-website
**Inconsistência:** "RP Consultoria" vs "RP One" — nomes diferentes entre funções

**Solução:** Criar tabela `company_portfolio` ou config JSONB no banco. Carregar dinamicamente.

**Status:** ❌ **Pendente**

---

### QUAL-002: Funções IA sem fallback
**Impacto:** Se o provedor principal cair, a função falha

| Função | Tem Fallback? |
|:---|:---:|
| analyze-website | ❌ (só OpenAI) |
| generate-post-caption | ❌ (só OpenAI) |
| summarize-meeting-prep | ❌ (só Lovable) |

**Solução:** Implementar dual-provider em todas as funções IA.

**Status:** ✅ **Parcialmente resolvido em `2026-04-05`**

| Função | Tem Fallback? |
|:---|:---:|
| analyze-website | ✅ `2026-04-05` (Lovable/Gemini adicionado) |
| generate-post-caption | ✅ `2026-04-05` (Lovable/Gemini adicionado) |
| summarize-meeting-prep | ⚠️ (só Lovable — OpenAI não disponível neste caso de uso) |

---

### QUAL-003: Respostas não-JSON em funções de análise
**Impacto:** Frontend precisa fazer parsing manual
- [[EF - analyze-linkedin-lead]] retorna `analysis` como string Markdown
- [[EF - analyze-website]] retorna `analysis` como string

**Solução:** Usar `response_format: { type: 'json_object' }` ou function calling.

**Status:** ❌ **Pendente**

---

## 🟢 Novas Funcionalidades Sugeridas

### NEW-001: Edge Function de Webhooks Inbound
**Descrição:** Receber webhooks de plataformas externas (Stripe, MercadoPago, n8n) com validação de assinatura.
**Benefício:** Automatizar processamento de pagamentos e eventos.
**Status:** ❌ Backlog

### NEW-002: Edge Function de Relatórios PDF
**Descrição:** Gerar relatórios PDF de performance de vendas, leads e métricas.
**Benefício:** Permitir que closers compartilhem relatórios com gestores.
**Status:** ❌ Backlog

### NEW-003: Edge Function de Lead Scoring Automático
**Descrição:** Calcular `close_probability` automaticamente baseado em interações, tempo no pipeline e segmento.
**Benefício:** Eliminar scoring manual e priorizar leads automaticamente.
**Status:** ❌ Backlog

### NEW-004: Cron Job para Geração de Embeddings
**Descrição:** Executar `generate-chat-embeddings` como cron (1x/dia) em vez de on-demand.
**Benefício:** Manter busca semântica sempre atualizada sem custo manual.
**Status:** ❌ Backlog

### NEW-005: Edge Function de Notificações Push
**Descrição:** Enviar notificações (email/push) quando leads mudam de status ou há inatividade.
**Benefício:** Alertas de follow-up perdido, leads esfriando.
**Status:** ❌ Backlog

### NEW-006: Rate Limiting por Tenant
**Descrição:** Implementar rate limiting por `user_id` nas funções IA.
**Benefício:** Prevenir abuso e custos inesperados de LLM.
**Status:** ❌ Backlog

---

## 🎨 Melhorias de CRM (Frontend)

### UI-001: Dashboard com filtro de período
**Atual:** Dashboard mostra métricas de todos os tempos.
**Melhoria:** Adicionar seletor de período (7d, 30d, 90d, YTD, custom).

### UI-002: Exportação de leads em CSV/Excel
**Atual:** Importação de CSV existe, mas não exportação.
**Melhoria:** Botão de exportar leads filtrados para CSV com todas as colunas.

### UI-003: Pipeline visual (Kanban de vendas)
**Atual:** Leads são visualizados em tabela.
**Melhoria:** Kanban drag-and-drop com colunas por status (como o de posts).

### UI-004: Histórico de alterações do lead
**Atual:** Sem audit trail.
**Melhoria:** Timeline de mudanças (quem alterou, quando, o quê) usando triggers SQL.

### UI-005: WhatsApp Web integrado
**Atual:** Conversas de WhatsApp são importadas via n8n.
**Melhoria:** Widget de chat in-app que envia mensagens diretamente via API.

### UI-006: Multi-user / Teams
**Atual:** Cada `user_id` é um tenant isolado.
**Melhoria:** Adicionar `organization_id` para equipes compartilharem leads.
**Referência:** [[Nexus - Decisões Técnicas (ADR)#ADR-003]]

### UI-007: Notificações in-app
**Atual:** Apenas `toast()` para feedback imediato.
**Melhoria:** Central de notificações persistente (novo lead atribuído, meta atingida, etc).

### UI-008: Dark mode toggle
**Atual:** Suporta dark mode via Tailwind mas sem toggle visível.
**Melhoria:** Botão no header com preferência salva em `localStorage`.

---

## 📊 Resumo de Prioridades

|    Prioridade     | Count | Categorias                            |            Progresso            |
| :---------------: | :---: | :------------------------------------ | :-----------------------------: |
|    🔴 Crítico     |   4+7 | SEC-001, SEC-002, BUG-001, BUG-002 + SEC-001..007 (review) | ✅ 11/11 |
|      🟠 Alto      |   4+5+5 | PERF-001, PERF-002, COST-001, LOG-001 + review findings | ✅ 18/18 (14/14 applied) |
|     🟡 Médio      |   3   | QUAL-001, QUAL-002, QUAL-003          | ⚠️ 1/3 parcial (QUAL-002)       |
| 🟢 Novo (Backend) |   6   | NEW-001 a NEW-006                     |              ❌ 0/6              |
| 🎨 CRM (Frontend) |   8   | UI-001 a UI-008                       |              ❌ 0/8              |

**Próxima sprint sugerida:**
1. **PERF-001** → implementar cache KV/Redis + summarization no nexus-assistant
2. **QUAL-001** → tabela `company_portfolio` ou config JSONB para portfólio dinâmico
3. **QUAL-003** → structured JSON output em analyze-linkedin-lead e analyze-website
4. **MOD-002** _(opcional)_ → adicionar testes unitários para os módulos do nexus-assistant
5. **NEW-001** → Edge Function de Webhooks Inbound (Stripe, MercadoPago)

> ✅ **Sprint 2 completa:** COST-001 e LOG-001 aplicados em **todas as 12 funções IA**. SEC-002 aplicado em todas as funções. nexus-assistant modularizado de 782 → 160 linhas.
>
> ✅ **Sprint de Revisão de Segurança completa:** 18 fix aplicados em 15 arquivos, incluindo 7 críticos (cross-tenant, SSRF, prompt injection) + 5 altos + 6 menores. Aguardando deploy no Supabase Dashboard.

> ⚠️ **Atenção:** Verificar se as tabelas `n8n_chat_histories_duplicate`, `marketing_posts`, `pre_lead_linkedin` e `pre_leads` possuem a coluna `user_id`. Se alguma não tiver, será necessária uma migration SQL antes de o filtro de tenant funcionar corretamente nessas tabelas.

---

## Notas Relacionadas

- [[Nexus - Edge Functions]] — Visão geral de todas as functions
- [[Nexus - Regras de Negócio e Segurança]] — Contexto de segurança
- [[Nexus - Decisões Técnicas (ADR)]] — Trade-offs por trás das decisões
- [[Nexus - Padrões e Convenções]] — Padrões a seguir nas melhorias
