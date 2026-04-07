---
tags: [nexus, edge-function, serper, local-results, maps, intelligence, credit-gate]
created: 2026-04-03
updated: 2026-04-06
status: "🟢 Produção"
category: "Busca & Inteligência Geográfica"
parent: "[[Nexus - Edge Functions]]"
---

# ⚡ search-leads

> Busca e qualificação de leads via Serper.dev com **pipeline de Inteligência Geográfica** para extração de estabelecimentos físicos (Google Maps) e validação de email com DNS-over-HTTPS.

| Campo | Valor |
|:---|:---|
| **Arquivo** | `supabase/functions/search-leads/index.ts` |
| **Linhas** | ~830+ |
| **API Externa** | Serper.dev (`google.serper.dev/search`) |
| **Validação Email** | DNS-over-HTTPS (`dns.google/resolve?type=MX`) |
| **Auth** | `requireAuth()` do `_shared/auth.ts` (Zero Trust) |
| **Crédito** | 1 crédito por requisição (via `consumeCredits`) |

## Input / Output

```typescript
// Input
{ segmento: string, localizacao: string, num?: number (1-50, padrão: 10) }

// Output
{
  ok: true,
  // Novo schema: leads enriquecidos
  leads: [{
    name, url,
    metadata: { phone, email, location, website, company_name, rating, reviews, social_profiles },
    intent_score: 'high' | 'medium' | 'low',
    nexus_context: string,
    is_local: boolean
  }],
  // Legacy: compat com EmailSearchResultsCard
  results: [{ title, link, snippet, emails: string[], validEmails: number }],
  // Maps: estabelecimentos físicos filtrados
  business_leads: [{
    company_name, category, full_address, phone (E.164),
    website, google_maps_id, rating_score, reviews_count,
    intent_score: 'high' | 'medium' | 'low', nexus_context
  }],
  total: number,
  totalBusinessLeads: number,
  noiseBlocked: number,  // quantos ruídos foram descartados
  query: string,
  quotaRemaining: number
}
```

## Pipeline de Execução (10+ etapas)

```
1. requireAuth(req) → 401 se JWT inválido
2. consumeCredits() → 402 se sem créditos
3. getSerperApiKey() → resolve por tenant ou fallback env
4. checkQuota() → 429 se cota diária excedida
5. Parse params + sanitização SEC-007
6. POST → google.serper.dev/search
7. incrementQuota()
8a. extractMapsLeads() → filtro Google Maps localResults
8b. transformSerperResults() → organic + local combinados
9. enrichWithValidEmails() → validação MX DNS
10. scoreIntents() → high/medium/low
11. logger.success() → log em api_logs
```

## Módulo de Inteligência Geográfica

### 8a. Maps Leads (`extractMapsLeads`)

Processa exclusivamente o objeto `localResults` do Serper (Google Maps).

**Filtros de Exclusão (Hard Filter):**
- `instagram.com`, `facebook.com`, `linkedin.com`, `gupy.io`, `catho.com.br`
- `infojobs`, `indeed`, `trampos.co`, `vagas.com.br`, `glassdoor`
- Qualquer URL com keywords de recrutamento

**Lógica de Inclusão:**
- Captura: `title` → company_name, `address`, `category`, `phoneNumber`, `website`, `rating`, `reviewsCount`, `placeId`
- Telefone normalizado para E.164 (`+55 DDD numero`)
- **Regra de Negócio**: descarta lead sem website E sem telefone (lead frio)
- Conta `noiseBlocked` para logging de ruídos filtrados

### 8b. Full Pipeline (`transformSerperResults`)

Combina `organic` + `localResults` com prioridade para locais:

1. **Filtro de Ruído**: `.gov`, `.edu`, sites de notícias, vagas
2. **Limpeza de Nome**: remove sufixos como "| Página Inicial", "— Wikipedia"
3. **Extração de Contatos**: emails, telefones, WhatsApp, LinkedIn, Instagram, Facebook
4. **Business Summary**: resumo 1 frase para Análise 360°

## Validação de Email (3 etapas)

1. **Limpeza** — remove extensões de imagem e domínios inválidos
2. **Sintaxe** — regex rigorosa, TLD 2+ chars
3. **MX Record** — DNS-over-HTTPS (Google) verifica registro MX

> [!note] Por que DNS-over-HTTPS?
> Edge Functions não permitem TCP bruto (porta 25). DNS-over-HTTPS é o mais próximo de validação real possível.

## Intent Scoring

| Sinal | Peso |
|:---|:---|
| Email válido (MX passed) | +2 |
| Telefone capturado | +2 |
| LinkedIn Company | +2 |
| WhatsApp link | +1 |
| Google Reviews (>0) | +1 |
| Resultado Local (Maps) | +1 |

- **≥ 5** → `high`
- **≥ 2** → `medium`
- **< 2** → `low`

## Controle de Cota

- **1000 requisições/dia** por usuário (tabela `serper_quota`)
- Retorna **429** com mensagem clara quando excedido

## Logging

Registra métricas em `api_logs` incluindo:
- `totalFound`: leads orgânicos encontrados
- `totalBusinessLeads`: estabelecimentos Maps capturados
- `noiseBlocked`: ruídos (redes sociais/recrutamento) filtrados
- `query`: query sanitizada enviada à Serper

## Frontend

- **EmailSearchResultsCard** — consome `data.results` (formato legacy)
- **Opportunities.tsx** — chama via `supabase.functions.invoke`
- Futuro: componente para `data.business_leads` (estabelecimentos Maps)
