---
tags: [nexus, frontend, pre-leads, oportunidades]
created: 2026-04-05
updated: 2026-04-05
status: 🟢 Produção
category: Frontend
parent: "[[Nexus - Edge Functions]]"
---

# 📋 PreLeadsSection

> Componente que exibe pré-leads descobertos pelo Nexus Scout (Google Maps e LinkedIn).

| Campo | Valor |
|:---|:---|
| **Arquivo** | `src/components/opportunities/PreLeadsSection.tsx` |
| **Componentes filhos** | `PreLeadCard`, `PreLeadDetailsModal`, `PreLeadsKanban`, `PreLeadsTableView` |

## Interface PreLead
```typescript
{
  id, nome, telefone, website, endereco, cidade, estado, pais,
  categoria, origem, query_origem, status, score, observacoes,
  criado_em, latitude, longitude, linkedin_url, user_id,
  emails: string[],
  email_valid: boolean,
  email_validation_status: 'passed'|'invalid'|'no_email'
}
```

## Status do Pre-Lead
| Status | Significado |
|:---|:---|
| `novo` | Recém-descoberto |
| `analisando` | Em análise/enriquecimento |
| `qualificado` | Lead viável |
| `descartado` | Lead descartado (ruim/inválido) |
| `convertido` | Convertido em lead real no CRM |

## Ações Disponíveis
| Ação | Localização | Custo |
|:---|:---|:---|
| Converter em Lead | Modal + Card | Via `handle_user_action` RPC |
| Descartar | Modal + Card (ícone X vermelho) | Gratuito |
| Atribuir a mim | Card | Gratuito |
| Editar dados | Modal | Gratuito |
| Enriquecer dados | Modal (5 créditos) | 5 créditos |
| Análise IA (LinkedIn/site) | Modal | Via Edge Function |
| Gerar mensagem WhatsApp | Modal | Via Edge Function |

## Funcionalidades da PreLeadCard
- Selo de validação de email: ✅ Verificado (verde), ⚠️ Inválido (amarelo), ✉️ Sem email (cinza)
- Botão rápido de descartar (X vermelho) — não precisa abrir modal
- Botão de atribuir a si mesmo
- Layout: nome, categoria, localização, telefone, website, score (se houver)

## Filtros Disponíveis
- Data início/fim
- Status (dropdown)
- Segmento/Categoria (busca por texto)
- Cidade (busca por texto)
- "Atribuídos a mim" (checkbox)
- Visualização: Cards, Tabela ou Kanban

→ [[Nexus - Edge Functions#search-leads]] | [[EF - PreLeadCard]]
