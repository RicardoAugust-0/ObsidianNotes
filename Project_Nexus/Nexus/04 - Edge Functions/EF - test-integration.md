---
tags: [nexus, edge-function, integracao, teste]
created: 2026-04-03
updated: 2026-04-05
status: 🟢 Produção
category: Sistema
parent: "[[Nexus - Edge Functions]]"
---

# ⚡ test-integration

> Testa a conectividade de **13 integrações diferentes** com uma única Edge Function.

| Campo | Valor |
|:---|:---|
| **Arquivo** | `supabase/functions/test-integration/index.ts` |
| **Linhas** | ~240 |
| **Auth** | JWT |

## Integrações Suportadas

| `integration_key` | API Testada | Endpoint |
|:---|:---|:---|
| `serper_api` | Serper.dev | `google.serper.dev/search` |
| `openai_api` | OpenAI | `api.openai.com/v1/models` |
| `slack_api` | Slack | `slack.com/api/auth.test` |
| `hubspot_api` | HubSpot CRM | `api.hubapi.com/crm/v3/objects/contacts` |
| `pipedrive_api` | Pipedrive | `{domain}.pipedrive.com/api/v1/users/me` |
| `clickup_api` | ClickUp | `api.clickup.com/api/v2/user` |
| `gemini_api` | Google Gemini | `generativelanguage.googleapis.com/v1beta/models` |
| `meta_graph_api` | Meta Graph | `graph.facebook.com/v23.0/me` |
| `whatsapp_api` | WhatsApp | `graph.facebook.com/v23.0/{phone_id}` |
| `apollo_io` | Apollo.io | `api.apollo.io/v1/auth/health` |
| `supabase_api` | Supabase | `{url}/rest/v1/` |
| `uazapi` | Uazapi | `{url}/status` |

## Padrão de Resposta
```typescript
{ ok: boolean, message: string }
```

## ⚠️ Observações
- Apollo.io está aqui como teste — mas não é usado no restante do sistema
- Google CSE foi removido (migrado para Serper.dev em 2026-04)

→ [[Nexus - Melhorias e Roadmap]]
