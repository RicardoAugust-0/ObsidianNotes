---
tags: [nexus, edge-function, ia, linkedin]
created: 2026-04-03
status: 🟢 Produção
category: IA & Análise
parent: "[[Nexus - Edge Functions]]"
---

# ⚡ analyze-linkedin-lead

> Análise comercial de um perfil do LinkedIn Scout. Gera cards estruturados para uso em vendas B2B.

| Campo | Valor |
|:---|:---|
| **Arquivo** | `supabase/functions/analyze-linkedin-lead/index.ts` |
| **Linhas** | 106 |
| **Modelos** | `gpt-4o-mini` → fallback `gemini-2.5-flash` |

## Input / Output
```typescript
// Request
{ name: string, title: string, snippet: string, link: string }

// Response
{ success: boolean, analysis: string } // análise em texto (cards markdown)
```

## Cards Gerados
Visão Geral, Nível de Decisão, Dores, Soluções Aderentes, Ângulo de Abordagem, Mensagem de Contato, Notas Comerciais.

## Portfólio (RP Consultoria)
Nexus, Financeiro, Melhoria de Processos, Automações, Consultoria Estratégica.

## ⚠️ Melhorias
- [x] ~~Retorna `analysis` como string~~ — já retornava objeto JSON estruturado (`{ success, analysis }`)
- [x] ~~Portfólio hardcoded ("RP One" vs "RP Consultoria")~~ — extraído para constante `PORTFOLIO` e nome corrigido para "RP Consultoria" (2026-04-08)
- [x] ~~Não loga em `api_logs`~~ — já implementado via `createLogger`

→ [[Nexus - Melhorias e Roadmap]]
