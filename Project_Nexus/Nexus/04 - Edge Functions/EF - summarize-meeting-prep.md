---
tags: [nexus, edge-function, ia, reunioes]
created: 2026-04-03
status: 🟢 Produção
category: IA & Análise
parent: "[[Nexus - Edge Functions]]"
---

# ⚡ summarize-meeting-prep

> Gera briefing de preparação para uma call de vendas com base nos dados da reunião + lead + chat WhatsApp.

| Campo | Valor |
|:---|:---|
| **Arquivo** | `supabase/functions/summarize-meeting-prep/index.ts` |
| **Linhas** | 155 |
| **Modelo** | `google/gemini-3-flash-preview` (Lovable Gateway, sem fallback OpenAI!) |
| **Tabelas** | `reunioes_sdr`, `leads`, `n8n_chat_histories_duplicate` |

## Input / Output
```typescript
// Request
{ reuniao_id: string, lead_id?: string }
// Response
{ contexto, pontos_chave[], estrategia: { gatilhos[], objecoes[], proposta_valor }, proximos_passos[] }
```

## Fluxo
1. Busca reunião em `reunioes_sdr`
2. (Opcional) Busca lead completo em `leads`
3. (Opcional) Busca últimas 50 msgs WhatsApp via `phone`
4. IA gera briefing estruturado em JSON

## ⚠️ Melhorias
- [x] ~~**Sem fallback OpenAI**~~ — fallback para `gpt-4o-mini` adicionado quando Lovable falha (2026-04-08)
- [x] ~~Não loga em `api_logs`~~ — já implementado via `createLogger`
- [x] ~~Sem consumo de créditos~~ — já implementado via `consumeCredits`

→ [[Nexus - Melhorias e Roadmap]]
