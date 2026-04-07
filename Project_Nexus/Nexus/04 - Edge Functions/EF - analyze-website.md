---
tags: [nexus, edge-function, ia, prospeccao]
created: 2026-04-03
status: 🟢 Produção
category: IA & Análise
parent: "[[Nexus - Edge Functions]]"
---

# ⚡ analyze-website

> Analisa o site de uma empresa e gera resumo comercial. Também gera mensagem WhatsApp de primeiro contato.

| Campo | Valor |
|:---|:---|
| **Arquivo** | `supabase/functions/analyze-website/index.ts` |
| **Linhas** | 114 |
| **Modelo** | `gpt-4o-mini` (somente OpenAI, sem fallback Lovable!) |

## Dois Modos de Operação

### Modo 1: Análise de Website
```typescript
// Request
{ websiteUrl: string, companyName: string }
// Response
{ analysis: string } // texto markdown com 5 seções
```
Processo: Fetch HTML → Strip tags/scripts → Trunca 8000 chars → GPT analisa

### Modo 2: Geração de Mensagem WhatsApp
```typescript
// Request  
{ companyName: string, generateMessage: true, analysis: string }
// Response
{ message: string }
```

## ⚠️ Melhorias
- [ ] **Sem fallback LLM** — se OpenAI cair, a função falha
- [ ] Scraping básico (strip HTML) — pode perder contexto de SPAs
- [ ] User-Agent hardcoded como Windows Chrome
- [ ] Não loga em `api_logs`
- [ ] Não consome créditos

→ [[Nexus - Melhorias e Roadmap]]
