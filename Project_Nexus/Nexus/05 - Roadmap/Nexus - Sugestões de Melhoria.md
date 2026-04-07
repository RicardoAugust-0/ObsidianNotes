---
tags:
  - nexus
  - sugestoes
  - produto
  - tecnico
created: 2026-04-05
status: 🔄 Em Andamento
parent: "[[Nexus - Index do Projeto]]"
---

# 💡 Nexus — Sugestões de Melhoria

> Ideias e sugestões técnicas e de produto identificadas após análise profunda do código, arquitetura e fluxos do [[Nexus - Index do Projeto|Nexus]]. Organizadas por **impacto** e **esforço de implementação**.

---

## 🏢 Estratégia & Produto

### SUG-001 — Portfólio Dinâmico (sem hardcode)

**Problema atual:** O portfólio da RP Consultoria está duplicado em 5+ funções IA com textos levemente divergentes (algumas dizem "RP Consultoria", outras "RP One"). Qualquer mudança de produto exige deploy de múltiplas funções.

**Solução proposta:**
- Criar tabela `company_portfolio` no banco:
  ```sql
  CREATE TABLE company_portfolio (
    id uuid PRIMARY KEY,
    name text NOT NULL,           -- "Nexus", "Valora", etc.
    description text,
    target_pains text[],          -- dores que resolve
    is_active boolean DEFAULT true
  );
  ```
- Functions IA buscam o portfólio antes de montar o prompt
- Admin edita pelo painel → todas as funções atualizadas instantaneamente

**Impacto:** 🟠 Alto (consistência + agilidade de produto)
**Esforço:** 🟡 Médio (1 migration + ajuste nas 5 functions)

---

### SUG-002 — Dashboard de Monitoramento de IA

**Problema atual:** `api_logs` existe mas não há UI para visualizar custos e performance das funções IA.

**Solução proposta:**
Criar uma aba "IA & Custos" no menu de Logs com:
- Gráfico de uso por função (últimos 7/30 dias)
- Custo estimado por modelo (tokens × preço OpenAI/Gemini)
- Tempo médio de resposta por função
- Erros recentes com detalhes
- Top usuários por consumo (visão admin)

**Impacto:** 🟠 Alto (observabilidade real dos custos de IA)
**Esforço:** 🟡 Médio (query + componente de gráfico)

---

### SUG-003 — Lead Score Automático

**Problema atual:** `close_probability` é preenchido manualmente pelos usuários. Não reflete dados objetivos do pipeline.

**Solução proposta:**
Criar Edge Function `calculate-lead-score` que calcula probabilidade baseado em:
- Tempo no pipeline (mais tempo = menor probabilidade)
- Número de interações (mais interações = maior probabilidade)
- Segmento do lead vs. taxa histórica de fechamento desse segmento
- Presence of budget + cargo de decisão

Disparo: Trigger do banco após `UPDATE leads` ou via cron diário.

**Impacto:** 🟢 Alto (elimina subjetividade, melhora priorização)
**Esforço:** 🔴 Alto (lógica de ML + trigger + UI de exibição)

---

### SUG-004 — Planos com Funcionalidades Bloqueadas

**Problema atual:** `PlanGate` existe no código mas não há mapeamento claro de "qual funcionalidade está em qual plano".

**Solução proposta:**
Definir uma matriz de features × planos no banco:
```sql
CREATE TABLE plan_features (
  plan_type plan_type NOT NULL,
  feature_key text NOT NULL,
  is_enabled boolean DEFAULT true,
  PRIMARY KEY (plan_type, feature_key)
);
```
- `feature_key`: `"nexus_scout"`, `"analytics_ml"`, `"whatsapp_templates"`, etc.
- `PlanGate` busca via `useFeatureFlags()` hook com cache de 5min

**Impacto:** 🟠 Alto (monetização clara, upsell natural)
**Esforço:** 🟡 Médio

---

## ⚙️ Infraestrutura & Backend

### SUG-005 — Cache de Contexto no Chat (PERF-001)

**Problema atual:** Cada mensagem no Nexus Chat recarrega 8 tabelas do banco + monta um prompt de 10K-50K tokens. Custo alto por request.

**Solução proposta:**
Implementar cache KV no Supabase Edge (ou Upstash Redis) no `data-fetcher.ts`:

```typescript
// Pseudocódigo
const cacheKey = `tenant_data:${userId}`;
const cached = await kv.get(cacheKey);

if (cached) return JSON.parse(cached);

const data = await fetchFromDatabase(supabase, userId);
await kv.set(cacheKey, JSON.stringify(data), { ex: 300 }); // TTL 5min
return data;
```

**Impacto:** 🔴 Alto (redução de 80%+ no latência e custo por mensagem)
**Esforço:** 🟡 Médio (`data-fetcher.ts` já está isolado, só adicionar camada de cache)

---

### SUG-006 — Webhooks Inbound para Automações

**Problema atual:** O sistema só tem fluxos outbound (Nexus envia dados). Não há como sistemas externos (n8n, Stripe, MercadoPago) notificar o Nexus.

**Solução proposta:**
Criar `webhook-inbound` Edge Function com:
- Validação de assinatura HMAC (por provider)
- Parser de eventos: `lead.created`, `payment.confirmed`, `conversation.received`
- Dispatcher para ações internas (atualizar lead, adicionar créditos, criar interação)

```
n8n → POST /webhook-inbound?source=n8n → processa evento → atualiza banco
Stripe → POST /webhook-inbound?source=stripe → adiciona créditos ao usuário
```

**Impacto:** 🟢 Alto (abre ecossistema de automações)
**Esforço:** 🔴 Alto (validação de assinaturas, parser de eventos, dispatcher)

---

### SUG-007 — Rate Limiting por Tenant

**Problema atual:** Um usuário pode fazer chamadas ilimitadas às funções IA (o controle de créditos protege do custo, mas não da frequência de chamadas por segundo).

**Solução proposta:**
Implementar sliding window rate limit no `_shared/auth.ts`:

```typescript
// Checar antes de processar a requisição
const allowed = await checkRateLimit(userId, functionName, {
  windowSeconds: 60,
  maxRequests: 10, // ex: máx 10 análises por minuto
});
if (!allowed) return new Response("Too Many Requests", { status: 429 });
```

Usando Upstash Redis com `INCR` + `EXPIRE` para contagem.

**Impacto:** 🟠 Alto (proteção contra abuso e picos de custo)
**Esforço:** 🟡 Médio

---

### SUG-008 — Tipos TypeScript Compartilhados (Monorepo Lite)

**Problema atual:** Os tipos do banco (gerados pelo Supabase) existem em `src/integrations/supabase/types.ts` mas não são acessíveis nas Edge Functions. As functions usam `any` extensivamente.

**Solução proposta:**
Criar `supabase/functions/_shared/database.types.ts` com uma versão simplificada dos tipos de negócio (sem depender de geração automática):

```typescript
// _shared/database.types.ts
export interface Lead {
  id: string;
  user_id: string;
  name: string;
  phone: string | null;
  status: string;
  // ...
}
```

**Impacto:** 🟡 Médio (segurança de tipos + autocomplete nas funções)
**Esforço:** 🟢 Baixo (só digitação, sem mudanças de lógica)

---

## 🎨 Experiência do Usuário

### SUG-009 — Modo Kanban para Leads

**Problema atual:** Leads são visualizados apenas em tabela. O status (`Novo → Qualificado → Call agendada → Em negociação → Fechado`) sugere um pipeline natural.

**Solução proposta:**
Adicionar toggle "Tabela / Kanban" na página de Leads com:
- Colunas por status (drag-and-drop para mover)
- Cards compactos com nome, segmento, probabilidade e next action
- Contador de leads por coluna

**Impacto:** 🟠 Alto (UX de pipeline de vendas é padrão do mercado)
**Esforço:** 🟡 Médio (`@dnd-kit` ou `react-beautiful-dnd` + nova view)

---

### SUG-010 — Notificações In-App Persistentes

**Problema atual:** O sistema usa apenas `toast()` para feedback imediato. Não há central de notificações — eventos passados são perdidos.

**Solução proposta:**
- Tabela `notifications` no banco (evento, user_id, lido, created_at)
- Trigger SQL cria notificação quando: lead é atribuído, meta é atingida, conquista desbloqueada
- Bell icon no header com badge de não-lidas
- Sidebar de notificações (últimas 20)
- `Realtime` subscription para notificações ao vivo

**Impacto:** 🟠 Alto (engajamento + awareness de atividade do time)
**Esforço:** 🔴 Alto (DB + triggers + UI + Realtime)

---

### SUG-011 — Import de Leads pelo Chat

**Problema atual:** Import CSV existe mas é uma tela separada. Um usuário SaaS-nativo esperaria poder dizer no chat: "Importa essa lista de leads para mim".

**Solução proposta:**
Adicionar suporte a upload de arquivo na interface do Nexus Chat. Quando o arquivo é `.csv`:
1. Frontend faz upload para Supabase Storage
2. Envia URL para o `nexus-assistant`
3. Function baixa, parseia, valida e insere os leads com `user_id` correto
4. Retorna relatório: "Importados: 45. Duplicados: 3. Erros: 1."

**Impacto:** 🟡 Médio (experiência conversacional completa)
**Esforço:** 🔴 Alto (upload + parse + validação + feedback)

---

### SUG-012 — Dark Mode Toggle Persistente

**Problema atual:** O dark mode funciona (Tailwind + `next-themes`) mas não há botão visível na UI. O usuário não sabe que pode mudar.

**Solução proposta:**
- Ícone sol/lua no header ou no menu de perfil
- Preferência salva via `next-themes` (já usa `localStorage` internamente)
- Transição suave via CSS `transition: background-color 200ms`

**Impacto:** 🟢 Baixo-médio (polish de UX)
**Esforço:** 🟢 Muito baixo (20 linhas de código)

---

## 📊 Resumo de Prioridades

| ID | Sugestão | Impacto | Esforço | Tipo | Status |
|:---|:---|:---:|:---:|:---:|:---:|
| SUG-005 | Cache de contexto no Chat | 🔴 Alto | 🟡 Médio | Infra | ✅ Sprint 03 |
| SUG-001 | Portfólio dinâmico | 🟠 Alto | 🟡 Médio | Produto | 🔜 Pendente |
| SUG-002 | Dashboard de custos IA | 🟠 Alto | 🟡 Médio | Produto | 🔜 Pendente |
| SUG-009 | Modo Kanban para Leads | 🟠 Alto | 🟡 Médio | UX | ✅ Já existia |
| SUG-006 | Webhooks Inbound | 🟢 Alto | 🔴 Alto | Infra | 🔜 Pendente |
| SUG-007 | Rate Limiting | 🟠 Alto | 🟡 Médio | Infra | ✅ Sprint 03 |
| SUG-003 | Lead Score Automático | 🟢 Alto | 🔴 Alto | Produto | 🔜 Pendente |
| SUG-004 | Planos com feature flags | 🟠 Alto | 🟡 Médio | Produto | 🔜 Pendente |
| SUG-010 | Notificações In-App | 🟠 Alto | 🔴 Alto | UX | 🔜 Pendente |
| SUG-008 | Tipos TypeScript compartilhados | 🟡 Médio | 🟢 Baixo | DX | ✅ Sprint 03 |
| SUG-011 | Import de Leads pelo Chat | 🟡 Médio | 🔴 Alto | UX | 🔜 Pendente |
| SUG-012 | Dark Mode Toggle | 🟢 Baixo | 🟢 Muito baixo | UX | ✅ Já existia |

---

> [!tip] Quick wins
> **SUG-012** pode ser feito em 20 minutos. **SUG-008** melhora a manutenibilidade sem risco. Ambos são ótimos para sprints curtas.

> [!important] Maior ROI
> **SUG-005** (cache) reduz latência e custo de IA dramaticamente. Como `data-fetcher.ts` já está isolado no módulo, a implementação é cirúrgica.

---

## Notas Relacionadas

- [[Nexus - Melhorias e Roadmap]] — Roadmap técnico formal (bugs + melhorias auditadas)
- [[Nexus - Decisões Técnicas (ADR)]] — Trade-offs das decisões de arquitetura
- [[Sprint 02 - Observabilidade & Modularização]] — Última sprint concluída
- [[Sprint 03 - Performance & Tipos]] — SUG-005, 007, 008 implementados
