---
title: "Entidade - Submissions"
tags:
  - fatesck
  - entidade
  - database
aliases:
  - Submissions
  - Submissões
---

# Entidade — Submissions

Entregas de atividades pelos alunos.

## Campos

| Campo | Tipo | Null | Descrição |
|-------|------|------|-----------|
| `id` | UUID (PK) | — | Identificador |
| `assignmentId` | UUID (FK → Assignments) | — | Atividade entregue |
| `studentId` | UUID (FK → Users) | — | Aluno que entregou |
| `submittedAt` | TIMESTAMP | — | Timestamp da entrega |
| `isLate` | BOOLEAN | — | Entregue após o prazo |
| `score` | DECIMAL(5,2) | ✓ | Nota atribuída |
| `feedback` | TEXT | ✓ | Feedback criptografado no cliente |
| `feedbackIv` | TEXT | ✓ | IV do feedback |
| `fileEncryptionKey` | TEXT | ✓ | Chave AES dos arquivos |
| `createdAt` | TIMESTAMP | — | Data de criação |
| `updatedAt` | TIMESTAMP | — | Última atualização |

## Índices

- Único composto em `assignmentId` + `studentId`
- Índice em `studentId`

## Referências

- [[03 - Modelo de Dados]]
- [[Entidade - Users]]
- [[Entidade - Assignments]]
