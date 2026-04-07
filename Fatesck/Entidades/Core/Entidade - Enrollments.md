---
title: "Entidade - Enrollments"
tags:
  - fatesck
  - entidade
  - database
aliases:
  - Enrollments
  - Matrículas
---

# Entidade — Enrollments

Associação N:M entre users e classes com dados de participação.

## Campos

| Campo | Tipo | Null | Descrição |
|-------|------|------|-----------|
| `id` | UUID (PK) | — | Identificador |
| `userId` | UUID (FK → Users) | — | Aluno |
| `classId` | UUID (FK → Classes) | — | Turma |
| `role` | ENUM(`aluno`, `monitor`) | — | Papel na turma |
| `joinedAt` | TIMESTAMP | — | Data de ingresso |
| `removedAt` | TIMESTAMP | ✓ | Data de remoção (se removido) |

## Índices

- Único composto em `userId` + `classId`
- Índice em `classId`

## Referências

- [[03 - Modelo de Dados]]
- [[Entidade - Users]]
- [[Entidade - Classes]]
