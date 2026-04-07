---
tags:
  - nexus
  - sprint
  - historico
created: 2026-04-04
status: ✅ Concluída
parent: "[[Nexus - Index do Projeto]]"
---

# 🏃 Sprint 01 — Infra & Segurança

> **Objetivo:** Tornar o sistema seguro e rentável antes de expandir funcionalidades. Resolver todos os itens críticos (🔴) e os de alto impacto (🟠) identificados na auditoria inicial.

---

## 📋 Escopo

| Item | Tipo | Descrição |
|:---|:---:|:---|
| SEC-001 | 🔴 | Filtro de tenant (`user_id`) em todas as queries |
| SEC-002 | 🔴 | Autenticação JWT obrigatória em todas as funções |
| BUG-001 | 🔴 | `create_pre_lead` não implementado em `process-action` |
| BUG-002 | 🟡 | Inconsistência de versão da Meta API (v18.0 vs v23.0) |
| COST-001 | 🟠 | Consumo de créditos ausente em funções IA |
| LOG-001 | 🟠 | Logging padronizado ausente em funções IA |
| PERF-001 | 🟠 | nexus-assistant carregava dados sem limites |
| PERF-002 | 🟠 | generate-chat-embeddings sem de-duplicação |

---

## ✅ Entregáveis

### Shared Helpers (`_shared/`)

| Arquivo | O que resolve |
|:---|:---|
| `auth.ts` | `requireAuth()` — extrai user_id do JWT e bloqueia chamadas não autenticadas |
| `logging.ts` | `createLogger()` — fábrica de logger padronizado para `api_logs` |
| `credits.ts` | `consumeCredits()` — tabela de custos configurável por função |

### Funções Ajustadas

| Função | Correções |
|:---|:---|
| `nexus-assistant` | SEC-001 · SEC-002 · COST-001 · PERF-001 |
| `analyze-leads-data` | SEC-001 · SEC-002 · COST-001 · LOG-001 |
| `analyze-opportunities` | SEC-001 · SEC-002 · COST-001 · LOG-001 |
| `generate-chat-embeddings` | SEC-001 · SEC-002 · PERF-002 · LOG-001 |
| `check-achievements` | SEC-002 (userId movido do body → JWT) · LOG-001 |
| `list-users` | SEC-002 (`requireAdmin()` adicionado) · LOG-001 |
| `process-action` | BUG-001 (`create_pre_lead` implementado) · LOG-001 |
| `test-integration` | BUG-002 (Meta API `v18.0` → `v23.0`) |

---

## 🔗 Referências

- [[Nexus - Melhorias e Roadmap]] — Status completo de todos os itens
- [[Sprint 02 - Observabilidade & Modularização]] — Continuação desta sprint
