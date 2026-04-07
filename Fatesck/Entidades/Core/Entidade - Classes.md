---
title: "Entidade - Classes"
tags:
  - fatesck
  - entidade
  - database
aliases:
  - Classes
  - Turmas
---

# Entidade — Classes

Turmas/disciplinas criadas pelos professores.

## Campos

| Campo | Tipo | Null | Descrição |
|-------|------|------|-----------|
| `id` | UUID (PK) | — | Identificador |
| `name` | VARCHAR(200) | — | Nome da turma |
| `description` | TEXT | ✓ | Descrição da disciplina |
| `color` | VARCHAR(7) | — | Cor visual (hex) |
| `accessCode` | VARCHAR(8) | — | Código de ingresso (auto-gerado) |
| `professorId` | UUID (FK → Users) | — | Professor criador |
| `archived` | BOOLEAN | — | Se a turma foi arquivada |
| `chatEnabled` | BOOLEAN | — | Chat habilitado |
| `chatGroupKey` | TEXT | ✓ | Chave AES do grupo (crip.) |
| `createdAt` | TIMESTAMP | — | Data de criação |
| `updatedAt` | TIMESTAMP | — | Última atualização |

## Relacionamentos

```
Classes
├─── User (professor)
├─── Enrollment[]      (alunos matriculados)
├─── Assignment[]      (atividades da turma)
├─── Message[]         (mensagens do chat de turma)
└─── File[]            (arquivos da turma)
```

## Índices

- Único em `accessCode`
- Índice em `professorId`

## Referências

- [[03 - Modelo de Dados]]
- [[Entidade - Users]] para o professor
- [[Entidade - Assignments]] para atividades
- [[Entidade - Enrollments]] para matrículas
