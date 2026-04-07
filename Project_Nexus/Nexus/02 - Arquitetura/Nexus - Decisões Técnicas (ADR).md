---
tags:
  - nexus
  - adr
  - decisoes
  - arquitetura
created: 2026-04-03
parent: "[[Nexus - Index do Projeto]]"
---

# 🎯 Nexus — Decisões Técnicas (ADR)

> **Architecture Decision Records** — Registro das decisões de design identificadas no código do [[Nexus - Index do Projeto|Nexus]], com contexto, consequências e trade-offs.

---

## ADR-001: Supabase como Backend Único (BaaS)

**Status:** ✅ Aceito (em produção)
**Data:** ~Out/2025 (primeiras migrations)

**Contexto:** O Nexus precisava de Auth, DB, Storage, Functions e Realtime. O time é pequeno.

**Decisão:** Usar **Supabase** como BaaS completo, sem servidor próprio.

**Consequências:**
- ✅ Zero infra para gerenciar (serverless)
- ✅ Auth, RLS, Realtime e Storage prontos
- ✅ Edge Functions em Deno (TypeScript)
- ⚠️ Vendor lock-in no Supabase
- ⚠️ Cold starts nas Edge Functions
- ⚠️ Limites de concorrência por plano Supabase

---

## ADR-002: Chat como Command Bus (IA Orquestradora)

**Status:** ✅ Aceito
**Data:** ~Fev/2026

**Contexto:** O sistema precisava de uma forma intuitiva de disparar ações complexas (busca LinkedIn, busca email, análise de leads) sem onboarding pesado.

**Decisão:** O Nexus Chat interpreta linguagem natural e emite **marcadores especiais** que o frontend intercepta para disparar ações programáticas.

**Marcadores implementados:**
- `[LINKEDIN_SEARCH_FLOW]` / `[LINKEDIN_COLLECTING_DATA]` / `[LINKEDIN_SEARCH_STARTED]` / `[SEARCH_PARAMS:{json}]`
- `[EMAIL_SEARCH_FLOW]` / `[EMAIL_COLLECTING_DATA]` / `[EMAIL_SEARCH_STARTED]` / `[EMAIL_SEARCH_PARAMS:{json}]`

**Consequências:**
- ✅ UX natural (conversar com a IA para executar ações)
- ✅ Extensível (adicionar novos marcadores = novas ações)
- ⚠️ Acoplamento entre resposta da IA e lógica do frontend
- ⚠️ Marcadores podem vazar na UI se não forem filtrados corretamente
- ⚠️ Latência dupla: 1 chamada para extração de params + 1 para busca

**Referência:** [[Nexus - Edge Functions#nexus-assistant]] | [[Nexus - Fluxo de Dados#2. Fluxo do Nexus Chat]]

---

## ADR-003: Isolamento por user_id (Não company_id)

**Status:** ✅ Aceito
**Data:** Mar/2026 (migration `saas_multitenant`)

**Contexto:** O modelo de multi-tenancy precisava definir o nível de isolamento.

**Decisão:** Cada `user_id` é um **tenant independente**. NÃO existe `company_id`. RLS filtra por `user_id = auth.uid()`.

**Consequências:**
- ✅ Simplicidade total de RLS
- ✅ Cada usuário é totalmente isolado
- ⚠️ Dificulta cenários de **equipe/empresa** (múltiplos SDRs vendo os mesmos leads)
- ⚠️ Evolução futura para multi-user org vai exigir refatoração significativa
- 📝 O docs `PLAN-saas-multitenant.md` menciona `company_id` como futuro, mas o schema atual não tem

**Referência:** [[Nexus - Regras de Negócio e Segurança#Isolamento Multi-Tenant]]

---

## ADR-004: Dual LLM Provider com Fallback

**Status:** ✅ Aceito
**Data:** ~Mar/2026

**Contexto:** Dependência de um único provedor LLM é arriscado (rate limits, custos, downtime).

**Decisão:** Implementar **dois provedores** com fallback automático:

| Prioridade | Provedor | Modelo |
|:---:|:---|:---|
| 1 | Lovable AI Gateway (Gemini) | `google/gemini-3-flash-preview` |
| 2 | OpenAI API | `gpt-4.1` / `gpt-4o-mini` |

**Consequências:**
- ✅ Resiliência contra downtime de um provedor
- ✅ Custos otimizados (Lovable Gateway incluído no plano Lovable)
- ⚠️ Respostas podem variar entre modelos (comportamento inconsistente)
- ⚠️ `analyze-lead-360` usa Gemini 2.5 Flash como fallback (modelo diferente do chat)

**Referência:** [[Nexus - Edge Functions#Dual-Model Strategy]]

---

## ADR-005: Zero Trust — Backend como Árbitro Final

**Status:** ✅ Aceito
**Data:** Mar/2026 (migration `saas_multitenant`)

**Contexto:** Gates no frontend melhoram UX mas podem ser bypassados. Créditos precisam ser validados no servidor.

**Decisão:** Implementar **validação dupla**:
1. **Frontend (UX):** `CreditGate`/`PlanGate` bloqueiam visualmente
2. **Backend (Security):** `process-action` valida email, créditos e ownership antes de executar

**Middleware chain no `process-action`:**
```
1. Auth (JWT válido) → 401
2. Email verificado → 403
3. Billing/créditos via require_billing_or_credits() → 402
4. Consume credits atomicamente → consume_credits()
5. Tenant ownership (user_id do lead = user autenticado) → 403
```

**Consequências:**
- ✅ Segurança real (não depende do frontend)
- ✅ Códigos HTTP semânticos (401, 402, 403)
- ⚠️ Nem todas as Edge Functions implementam a chain completa
- 📝 `nexus-assistant` NÃO consome créditos (é a função mais chamada)

**Referência:** [[Nexus - Regras de Negócio e Segurança#Economia de Créditos]]

---

## ADR-006: Admin Hardcoded (CEO Email)

**Status:** ⚠️ Aceito (com ressalva)
**Data:** Mar/2026

**Contexto:** O sistema precisa de um admin padrão que tenha acesso total desde o signup.

**Decisão:** O trigger `handle_new_user()` verifica se o email é `rhyanpaablo@gmail.com` e atribui `role: 'admin'` automaticamente.

```sql
IF new_email = 'rhyanpaablo@gmail.com' THEN
  v_role := 'admin'::public.user_role;
END IF;
```

**Consequências:**
- ✅ O CEO sempre tem acesso total, mesmo se o banco for recriado
- ⚠️ **Email hardcoded** no banco — se mudar o email, perde o admin
- ⚠️ Não escala para múltiplos admins (precisa grant manual no DB)
- 📝 Sugestão futura: tabela `admin_emails` ou env var

---

## ADR-007: React Query como Único State Manager

**Status:** ✅ Aceito
**Data:** Desde o início

**Contexto:** O projeto precisa gerenciar dados do servidor (leads, conversas, templates) e estado local (UI).

**Decisão:** **Não usar** Redux, Zustand, MobX ou Context API. Usar apenas:
- **TanStack React Query** para server state
- **useState** para estado local de componentes
- **Supabase Realtime** para atualizações push

**Consequências:**
- ✅ Menos boilerplate que Redux
- ✅ Cache automático com invalidação
- ✅ Sem prop drilling excessivo
- ⚠️ Estado "global" como `currentUser` precisa ser re-fetched em cada hook que precisa
- ⚠️ `useUserRole` é chamado em muitos componentes (pode causar re-renders)

**Referência:** [[Nexus - Padrões e Convenções#5. Padrão de Cache]]

---

## ADR-008: SSE para Streaming de Chat (Não WebSocket)

**Status:** ✅ Aceito
**Data:** ~Jan/2026

**Contexto:** O chat IA precisa mostrar tokens incrementalmente (efeito "typing").

**Decisão:** Usar **Server-Sent Events (SSE)** nativamente via `ReadableStream`, compatível com a API da OpenAI.

**Formato:**
```
data: {"choices":[{"delta":{"content":"..."}}]}
data: {"choices":[{"delta":{"content":"..."}}]}
data: [DONE]
```

**Consequências:**
- ✅ Compatível com formato padrão OpenAI (zero parsing customizado)
- ✅ Unidirecional (servidor → cliente), mais simples que WebSocket
- ✅ Funciona com `fetch()` nativo (sem lib adicional)
- ⚠️ Sem bidireção (não permite "cancelar" stream do cliente)
- ⚠️ SSE não suporta payload binário

**Referência:** [[Nexus - Padrões e Convenções#Padrão de Streaming SSE]]

---

## ADR-009: Serper.dev para Busca de Emails (Não Apollo/Hunter)

**Status:** ✅ Aceito (migrou do Google CSE em 2026-04)
**Data:** ~Fev/2026

**Contexto:** O sistema precisava de busca de emails B2B para prospecção.

**Decisão:** Inicialmente usar Google Custom Search Engine API. Migrou para **Serper.dev** em 2026-04.

**Lógica de busca:**
```
Query: "{segmento}" "{localização}" ("e-mail" OR "contato" OR "@")
→ Extrai emails dos snippets via regex
→ Filtra extensões de imagem e domínios inválidos
```

**Consequências:**
- ✅ Custo baixo (Serper tem plano generoso)
- ✅ Sem dependência de serviço B2B externo
- ⚠️ Qualidade dos emails variável (depende do que aparece nos snippets)
- ⚠️ Não valida se o email é real (sem verificação SMTP)

**Referência:** [[Nexus - Edge Functions#search-leads]]

---

## ADR-010: Dados n8n como Legado Integrado

**Status:** 🔄 Em transição
**Data:** Desde antes do SaaS

**Contexto:** O sistema tinha automações n8n que persistiam chat de WhatsApp em tabelas próprias.

**Decisão:** Manter as tabelas `n8n_chat_histories*` e integrá-las ao Nexus (a IA consulta essas conversas na análise 360° e no contexto do chat).

**Consequências:**
- ✅ Histórico rico de conversas WhatsApp disponível para a IA
- ✅ Vinculação de chats a leads via `n8n_chat_histories_duplicate.lead_id`
- ⚠️ Tabelas com nomes inconsistentes (`_duplicate`, `_servix`)
- ⚠️ Dados legados que podem ter formato diferente
- 📝 Considerar migração para tabela unificada no futuro

---

## Notas Relacionadas

- [[Nexus - Arquitetura]] — Resultado das decisões no sistema
- [[Nexus - Padrões e Convenções]] — Como as decisões se manifestam no código
- [[Nexus - Edge Functions]] — Implementação das decisões de IA
- [[Nexus - Regras de Negócio e Segurança]] — Regras derivadas das decisões
- [[Nexus - Modelo de Dados e Banco]] — Schema resultante das decisões
