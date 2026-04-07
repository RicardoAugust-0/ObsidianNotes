---
tags: [nexus, edge-function, ia, chat]
created: 2026-04-03
status: 🟢 Produção
category: IA & Análise
parent: "[[Nexus - Edge Functions]]"
---

# ⚡ nexus-assistant

> **O cérebro do Nexus.** Motor principal do chat IA com streaming.

| Campo | Valor |
|:---|:---|
| **Arquivo** | `supabase/functions/nexus-assistant/index.ts` |
| **Linhas** | 751 (maior do projeto) |
| **Runtime** | Deno (`serve()`) |
| **Auth** | JWT via header + `getApiKeys()` para tenant keys |
| **Logging** | `api_logs` (duração, modelo, tokens) |

---

## Input / Output

```typescript
// Request
{ message: string, history: Message[], stream: boolean }

// Response (stream=false)
{ response: string }

// Response (stream=true)
SSE: data: {"choices":[{"delta":{"content":"..."}}]}\n\n
      data: [DONE]\n\n
```

---

## Modelos Utilizados

| Contexto | Modelo | Provedor |
|:---|:---|:---|
| Chat principal (com LOVABLE_KEY) | `google/gemini-3-flash-preview` | Lovable Gateway |
| Chat principal (fallback) | `gpt-4.1` | OpenAI |
| Extração de params LinkedIn/Email | `google/gemini-3-flash-preview` ou `gpt-4o-mini` | Lovable/OpenAI |
| Busca semântica (embeddings) | `text-embedding-ada-002` | OpenAI |

---

## Fluxos Internos

### 1. LinkedIn Scout Flow
- **Trigger:** message contém `[LINKEDIN_SEARCH_FLOW]`
- **Processo:** Coleta dados (segmento, cidade, cargo) → LLM extrai params → Emite `[LINKEDIN_SEARCH_STARTED][SEARCH_PARAMS:{json}]`
- **Frontend intercepta** o marcador e dispara `useLinkedInSearch`

### 2. Email Search Flow
- **Trigger:** message contém `[EMAIL_SEARCH_FLOW]`
- **Processo:** Coleta dados (segmento, localização) → LLM extrai params → Emite `[EMAIL_SEARCH_STARTED][EMAIL_SEARCH_PARAMS:{json}]`
- **Frontend intercepta** e dispara `useEmailSearch`

### 3. Consulta de Lead
- **Trigger:** message contém `"Consultar lead:"`
- **Processo:** Busca lead no banco + `n8n_chat_histories_duplicate` (últimas 20 msgs) + `lead_ai_analyses` (últimas 5) → Monta contexto rico

### 4. Busca Semântica
- **Trigger:** Palavras-chave: "conversa", "disse", "falou", "mencionou", "padrão", "buscar"
- **Processo:** Gera embedding → `match_documents()` RPC → top 10 resultados por similaridade

### 5. Comandos Rápidos
| Comando | System Prompt |
|:---|:---|
| `/metricas` | Métricas executivas |
| `/leads` | Informações detalhadas |
| `/oportunidades` | Pipeline |
| `/preleads` | Pré-leads prospectados |
| `/posts` | Marketing |

### 6. Chat Geral
Carrega **8 tabelas em paralelo** a cada request: leads (300), oportunidades (100), chats WhatsApp (1000), posts (100), LinkedIn (200), Google Maps (200), metas (50), conquistas (all).

---

## ⚠️ Melhorias Sugeridas

- [ ] **Performance crítica**: Carrega milhares de registros a cada request — implementar cache ou filtro por tenant
- [ ] **Consumo de créditos**: Não consome créditos (a função mais chamada deveria?)
- [ ] **Prompt muito longo**: O system prompt pode ultrapassar 100K tokens — risco de custo alto
- [ ] **Sem rate limiting por tenant**: Qualquer usuário pode fazer requests ilimitados

→ [[Nexus - Melhorias e Roadmap#EF - nexus-assistant]]

## ⚡ Atualizações de Segurança (2026-04-05)

**Auth obrigatória:** Acesso anônimo removido (SEC-001). `requireAuth()` agora obrigatório, sem try/catch.

**Tenant isolation garantido:** `fetchTenantData()` nunca executa sem `userId` (SEC-002). Todas as 8 queries têm `.eq("user_id", userId)`.

**Prompt injection prevention:** Dados do usuário envoltos em tags XML (`<lead_card>`, `<opportunity_card>`, etc.) com `escapeForPrompt()` que remove `<>` e trunca a 500 chars (SEC-004).

**Credit consumption ativo:** Nexus-assistant consome 1 crédito por request (implementado desde sprint anterior).

**JSON.parse seguro:** Flows LinkedIn e Email com try/catch em JSON.parse de resposta LLM.

**Import path corrigido:** `linkedin.ts` agora importa `"../types.ts"` em vez de `"./types.ts"`.

---

## Notas Relacionadas
- [[Nexus - Edge Functions]] — Visão geral de todas as functions
- [[Nexus - Fluxo de Dados]] — Diagramas de fluxo
- [[Nexus - Decisões Técnicas (ADR)#ADR-002]] — Decisão do Command Bus
