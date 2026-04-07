---
tags: [nexus, edge-function, marketing, imagem]
created: 2026-04-03
status: 🟢 Produção
category: Conteúdo & Mídia
parent: "[[Nexus - Edge Functions]]"
---

# ⚡ generate-brand-image

> Gera imagens de marca para redes sociais. Suporta posts únicos e slides de carrossel.

| Campo | Valor |
|:---|:---|
| **Arquivo** | `supabase/functions/generate-brand-image/index.ts` |
| **Linhas** | 130 |
| **Modelos** | `gemini-2.5-flash-image-preview` (Lovable) → fallback `dall-e-3` (OpenAI) |

## Input / Output
```typescript
{ theme: string, colors?: string[], logoUrl?: string, isCarousel?: boolean, carouselIndex?: number }
→ { imageUrl: string, provider: string }
```

## Diferencial
- Suporta **paleta de cores da marca** (personalizado por post)
- Gera slides individuais para carrossel (índice 0/1/2)
- Retry com backoff exponencial (até 3 tentativas no fallback OpenAI)
- `1024x1024` no DALL-E 3

## ⚠️ Melhorias
- [ ] Não loga em `api_logs`
- [ ] Imagem DALL-E retorna URL temporária — deveria salvar no Supabase Storage

→ [[Nexus - Melhorias e Roadmap]]
