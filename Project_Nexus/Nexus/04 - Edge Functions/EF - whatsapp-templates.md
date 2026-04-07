---
tags: [nexus, edge-function, whatsapp, meta]
created: 2026-04-03
status: 🟢 Produção
category: Busca & Integração
parent: "[[Nexus - Edge Functions]]"
---

# ⚡ whatsapp-templates

> CRUD completo de templates WhatsApp Business via Meta Graph API v23.0.

| Campo | Valor |
|:---|:---|
| **Arquivo** | `supabase/functions/whatsapp-templates/index.ts` |
| **Linhas** | 435 |
| **API Externa** | Meta Graph API v23.0 (`graph.facebook.com/v23.0`) |
| **Auth** | JWT + credenciais em `integration_configs` (whatsapp_api) |

## Ações Disponíveis
| Action | Método Meta | Endpoint |
|:---|:---:|:---|
| `list` / `sync` | GET | `/{waba_id}/message_templates` |
| `create` | POST | `/{waba_id}/message_templates` |
| `update` | POST | `/{template_id}` |
| `delete` | DELETE | `/{waba_id}/message_templates?name=...` |

## Validações
- Nome: `^[a-z][a-z0-9_]{0,511}$`
- Categorias: `MARKETING`, `UTILITY`, `AUTHENTICATION`
- Template ID: `^\d+$`

## Validações de Segurança (Sprint 06)
- Delete agora valida `name` com `TEMPLATE_NAME_REGEX` (SAN-001)
- Error messages genéricas no catch — sem vazamento de tokens (ERR-001)

## Error Mapping (Meta → PT-BR)
| Código Meta | HTTP | Mensagem |
|:---:|:---:|:---|
| 190 | 401 | Token expirado |
| 4 | 429 | Rate limit |
| 100 | 400 | Parâmetro inválido |

→ [[Nexus - Melhorias e Roadmap]]
