---
title: "Entidade - Assignments"
tags:
  - fatesck
  - entidade
  - database
aliases:
  - Assignments
  - Atividades
---

# Entidade — Assignments

Atividades/avaliações criadas pelo professor para uma turma.

## Campos

| Campo | Tipo | Null | Descrição |
|-------|------|------|-----------|
| `id` | UUID (PK) | — | Identificador |
| `title` | VARCHAR(300) | — | Título da atividade |
| `description` | TEXT | ✓ | Instruções (não criptografado) |
| `classId` | UUID (FK → Classes) | — | Turma associada |
| `creatorId` | UUID (FK → Users) | — | Professor que criou |
| `dueDate` | TIMESTAMP | — | Prazo de entrega |
| `maxScore` | DECIMAL(5,2) | — | Nota máxima (ex: 10.00) |
| `weight` | DECIMAL(3,2) | — | Peso na nota final (ex: 1.50) |
| `allowLate` | BOOLEAN | — | Permite envio após prazo |
| `isPublished` | BOOLEAN | — | Visível para alunos |
| `fileEncryptionKey` | TEXT | ✓ | Chave AES dos anexos (crip.) |
| `createdAt` | TIMESTAMP | — | Data de criação |
| `updatedAt` | TIMESTAMP | — | Última atualização |

## Relacionamentos

```
Assignments
├─── Class
├─── User (creator)
├─── Submission[]   (entregas dos alunos)
└─── File[]         (anexos do professor)
```

## Índices

- Índice em `classId`
- Índice em `creatorId`
- Índice em `dueDate` (ordenação por prazo)
- Índice composto em `classId` + `isPublished` + `dueDate`

## Referências

- [[03 - Modelo de Dados]]
- [[Entidade - Submissions]] para entregas
- [[Entidade - Classes]] para turmas
