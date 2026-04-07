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
- [ ] **Sem fallback OpenAI** — se Lovable cair, função falha
- [ ] Não loga em `api_logs`
- [ ] Sem consumo de créditos

→ [[Nexus - Melhorias e Roadmap]]
