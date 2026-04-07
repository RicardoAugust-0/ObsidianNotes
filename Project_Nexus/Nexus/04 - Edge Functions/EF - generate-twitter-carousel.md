---
tags: [nexus, edge-function, marketing, carrossel]
created: 2026-04-03
status: 🟢 Produção
category: Conteúdo & Mídia
parent: "[[Nexus - Edge Functions]]"
---

# ⚡ generate-twitter-carousel

> Gera carrosséis no estilo Twitter Card minimalista para Instagram (5-7 slides).

| Campo | Valor |
|:---|:---|
| **Arquivo** | `supabase/functions/generate-twitter-carousel/index.ts` |
| **Linhas** | 103 |
| **Modelos** | `gpt-4o-mini` → fallback `gemini-2.5-flash` |

## Input / Output
```typescript
{ tema?: string, url?: string, handle?: string, avatarUrl?: string }
→ { slides: [{ titulo: string, texto: string, imagem_prompt: string }] }
```

## Diferencial
- Aceita URL para scraping de conteúdo (strip HTML, trunca 5000 chars)
- Estilo visual: fundo branco, texto preto, avatar circular, seta (→) no canto

→ [[Nexus - Melhorias e Roadmap]]
