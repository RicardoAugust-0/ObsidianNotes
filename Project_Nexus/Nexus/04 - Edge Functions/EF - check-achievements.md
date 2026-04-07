---
tags: [nexus, edge-function, gamificacao]
created: 2026-04-03
status: 🟢 Produção
category: Sistema
parent: "[[Nexus - Edge Functions]]"
---

# ⚡ check-achievements

> Verifica se o usuário atingiu critérios para novas conquistas e atribui automaticamente.

| Campo | Valor |
|:---|:---|
| **Arquivo** | `supabase/functions/check-achievements/index.ts` |
| **Linhas** | 132 |
| **Auth** | Sem auth! Recebe `userId` no body |
| **Tabelas** | `achievements`, `user_achievements`, `leads`, `lead_projects` |

## Input / Output
```typescript
{ userId: string }
→ { success: boolean, newAchievements: Achievement[],
    stats: { closedLeads, totalLeads, conversionRate, totalRevenue } }
```

## Conquistas Implementadas
| Nome | Critério |
|:---|:---|
| 5 Vendas | `closedLeads >= 5` |
| 10 Vendas | `closedLeads >= 10` |
| Vendedor Top | `closedLeads >= 25` |
| Mestre das Vendas | `closedLeads >= 50` |
| Taxa de Conversão | `conversionRate >= 50%` |
| Receita 100K | `totalRevenue >= R$ 100.000` |

## ⚠️ Melhorias
- [ ] **🔴 SEGURANÇA**: Sem validação JWT — qualquer pessoa pode chamar com qualquer userId
- [ ] Conquistas verificadas por **nome hardcoded** (string matching) — frágil
- [ ] Sem critério dinâmico — deveria usar campo `criteria` da tabela `achievements`

→ [[Nexus - Melhorias e Roadmap]]
