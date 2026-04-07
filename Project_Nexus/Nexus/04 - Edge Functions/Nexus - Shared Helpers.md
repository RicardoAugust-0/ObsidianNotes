---
tags:
  - nexus
  - backend
  - shared-helpers
  - arquitetura
created: 2026-04-04
updated: 2026-04-05
parent: "[[Nexus - Edge Functions]]"
---

# 🧰 Nexus — Shared Helpers (`_shared/`)

> Módulos utilitários compartilhados entre todas as Edge Functions. Criados na [[Sprint 01 - Infra & Segurança]]. Importados via `import { ... } from "../_shared/modulo.ts"`.

---

## Estrutura

```
supabase/functions/_shared/
├── auth.ts      → requireAuth(), requireAdmin()
├── logging.ts   → createLogger(), logApiCall()
├── credits.ts   → consumeCredits()
└── cors.ts      → corsHeaders (legado, mantido para compatibilidade)
└── rate-limit.ts → checkRateLimit() por tenant
```

---

## `auth.ts` — Autenticação e Autorização

### `requireAuth(req: Request)`

Extrai e valida o JWT do header `Authorization: Bearer`. Retorna `{ user, supabase }`.

```typescript
// Uso em Edge Functions
const { user, supabase } = await requireAuth(req);
// user.id → user_id para filtros de tenant
```

**Fluxo:**
1. Extrai header `Authorization`
2. Chama `supabase.auth.getUser(token)`
3. Se inválido → lança `Response` com `401 Unauthorized`
4. Se válido → retorna `{ user, supabase }` (supabase autenticado como service role)

### `requireAdmin(req: Request)`

Extende `requireAuth()` com verificação de role admin.

```typescript
const { user } = await requireAdmin(req);
// Lança 403 se user não for admin
```

### `corsHeaders`

```typescript
export const corsHeaders = {
  "Access-Control-Allow-Origin": "*",
  "Access-Control-Allow-Headers": "authorization, x-client-info, apikey, content-type",
};
```

### `errorResponse(message, status = 500)`

Helper para retornar erros padronizados com CORS.

---

## `logging.ts` — Logging Padronizado

### `createLogger(supabase, functionName, userId)`

Fábrica que retorna um objeto `{ success, error }` para logging em `api_logs`.

> Atualizado em 2026-04-05 **(SEC-012)**: Metadata agora sanitizada antes de gravar:
> - **Strip keys sensíveis**: `apiKey`, `secret`, `token`, `password`, `authorization` são removidos/ocultados
> - **Truncate**: strings limitadas a 1024 chars para prevenir payloads maliciosos
> - **Sanitização recursiva**: aplica-se a objetos aninhados JSON

```typescript
const logger = createLogger(supabase, "analyze-lead-360", user.id);

// No caminho de sucesso:
await logger.success({
  duration_ms: Date.now() - startTime,
  model: "gpt-4o-mini",
  prompt_tokens: data.usage?.prompt_tokens,
  completion_tokens: data.usage?.completion_tokens,
  total_tokens: data.usage?.total_tokens,
  metadata: { lead_id: lead.id, provider: "openai" },
});

// No caminho de erro:
await logger.error({
  duration_ms: Date.now() - startTime,
  error_message: "Mensagem do erro",
});
```

**Schema da tabela `api_logs`:**

| Coluna | Tipo | Descrição |
|:---|:---|:---|
| `function_name` | text | Nome da Edge Function |
| `user_id` | uuid | Tenant que fez a chamada |
| `status` | text | `"success"` ou `"error"` |
| `model` | text | Modelo de IA utilizado |
| `prompt_tokens` | int | Tokens de entrada |
| `completion_tokens` | int | Tokens de saída |
| `total_tokens` | int | Soma |
| `duration_ms` | int | Tempo de execução |
| `error_message` | text | Mensagem de erro (quando aplicável) |
| `metadata` | jsonb | Dados contextuais livres |

### `logApiCall(supabase, data)` *(legado)*

Versão baixo nível. Preferir `createLogger()` nas novas implementações.

---

## `credits.ts` — Controle de Custos

### `consumeCredits(supabase, userId, functionName)`

Verifica e debita créditos antes de processar a requisição. Retorna `null` se OK, ou uma `Response` de erro se bloqueado.

```typescript
const creditError = await consumeCredits(supabase, user.id, "analyze-lead-360");
if (creditError) return creditError; // Retorna 402 se sem créditos
```

**Tabela de custos configurada:**

| Função | Créditos |
|:---|:---:|
| `nexus-assistant` | 1 |
| `analyze-lead-360` | 2 |
| `analyze-leads-data` | 3 |
| `analyze-opportunities` | 2 |
| `analyze-linkedin-lead` | 1 |
| `analyze-website` | 1 |
| `generate-message-suggestions` | 1 |
| `summarize-chat-history` | 1 |
| `summarize-meeting-prep` | 1 |
| `generate-post-caption` | 1 |
| `generate-brand-image` | 3 |
| `generate-twitter-carousel` | 5 |
| `generate-chat-embeddings` | 5 |
| `search-leads` | 1 |

> Atualizado em 2026-04-05: adicionados `generate-chat-embeddings` (5 créditos) e `search-leads` (1 crédito). Mensagens de erro internas sanitizadas para não vazar detalhes ao cliente (SEC-010). Logs agora truncam strings e removem keys sensíveis (SEC-012). Mensagens de erro internas no `catch` principal também são sanitizadas com mensagem genérica (SEC-013). Logs truncados a 1024 chars para prevenir payloads maliciosos.

> [!note] Admin bypass
> A RPC `consume_credits()` no banco verifica `is_admin()` antes de debitar. Admins nunca ficam sem créditos.

### `checkRateLimit(supabase, userId, options?)`

Verifica se o tenant ultrapassou o limite de requests em uma janela de tempo.

> Atualizado em 2026-04-05 **(SEC-011)**: Parâmetro `supabase` agora tipado como `SupabaseClient` em vez de `any`, garantindo type safety e autocomplete correto.

---

## Padrão de Uso Completo

Template mínimo para qualquer nova Edge Function IA:

```typescript
import { requireAuth, corsHeaders, errorResponse } from "../_shared/auth.ts";
import { createLogger } from "../_shared/logging.ts";
import { consumeCredits } from "../_shared/credits.ts";

Deno.serve(async (req) => {
  if (req.method === "OPTIONS") return new Response(null, { headers: corsHeaders });

  const startTime = Date.now();

  try {
    const { user, supabase } = await requireAuth(req);
    const logger = createLogger(supabase, "nome-da-funcao", user.id);

    const creditError = await consumeCredits(supabase, user.id, "nome-da-funcao");
    if (creditError) return creditError;

    // ... lógica de negócio ...

    await logger.success({ duration_ms: Date.now() - startTime, model: "..." });
    return new Response(JSON.stringify({ data }), { headers: { ...corsHeaders, "Content-Type": "application/json" } });

  } catch (error) {
    if (error instanceof Response) return error;
    return errorResponse(error instanceof Error ? error.message : "Unknown error");
  }
});
```

---

## Notas Relacionadas

- [[Nexus - Edge Functions]] — Overview de todas as functions
- [[Sprint 01 - Infra & Segurança]] — Sprint que criou estes helpers
- [[Nexus - Regras de Negócio e Segurança]] — Contexto de segurança Zero Trust
