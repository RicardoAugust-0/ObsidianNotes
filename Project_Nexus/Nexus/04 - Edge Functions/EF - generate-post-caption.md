---
tags: [nexus, edge-function, marketing]
created: 2026-04-03
status: 🟢 Produção
category: Conteúdo & Mídia
parent: "[[Nexus - Edge Functions]]"
---

# ⚡ generate-post-caption

> Gera captions profissionais para posts de Instagram usando copywriting avançado.

| Campo | Valor |
|:---|:---|
| **Arquivo** | `supabase/functions/generate-post-caption/index.ts` |
| **Linhas** | 93 |
| **Modelo** | `gpt-4o-mini` (somente OpenAI, sem fallback) |

## Input / Output
```typescript
{ theme: string, prompt?: string, isCarousel?: boolean }
→ { caption: string }
```

## Técnicas no Prompt
AIDA, PAS, Storytelling, gatilhos mentais. Máximo 2200 chars. 3-5 hashtags. Tom PT-BR.

## ⚠️ Melhorias
- [ ] Sem fallback LLM
- [ ] Não loga em `api_logs`

→ [[Nexus - Melhorias e Roadmap]]
