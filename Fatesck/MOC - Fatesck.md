---
title: "Fatesck — Índice"
tags:
  - fatesck
  - moc
aliases:
  - Mapa do Projeto Fatesck
---

# Fatesck

> [!abstract] Sobre o Projeto
> Plataforma educacional com **criptografia de ponta a ponta (E2E)** para comunicação entre alunos e professores, gestão de turmas e entrega de atividades. Inspirado no Microsoft Teams, mas com princípio **zero-knowledge**.

---

## Planejamento

| Categoria | Notas |
|-----------|-------|
| **Visão Geral** | [[00 - Visão Geral]] |
| **Arquitetura** | [[01 - Arquitetura]] |
| **Segurança E2E** | [[02 - Segurança E2E]] |
| **Modelo de Dados** | [[03 - Modelo de Dados]] |
| **Stack Tecnológica** | [[04 - Stack Tecnológica]] |
| **Infraestrutura** | [[05 - Infraestrutura]] |

## Requisitos

| Categoria          | Notas              |
| ------------------ | ------------------ |
| **Funcionais**     | [[Requisitos Funcionais]]     |
| **Não Funcionais** | [[Requisitos Não Funcionais]] |

## Entidades do Banco de Dados

### Core
| Entidade | Nota |
|----------|------|
| Usuários | [[Entidade - Users]] |
| Turmas | [[Entidade - Classes]] |
| Matrículas | [[Entidade - Enrollments]] |

### Acadêmico
| Entidade | Nota |
|----------|------|
| Atividades | [[Entidade - Assignments]] |
| Submissões | [[Entidade - Submissions]] |

> *A modularizar demais: `Messages`, `Files`, `UserKeys`, `RefreshTokens`, `AuditLogs`*

## Segurança

| Categoria | Notas |
|-----------|-------|
| **Fluxo — Mensagem E2E** | [[Fluxo - Mensagem E2E]] |
| **Modelo de Chaves** | [[Modelo de Chaves]] |
| **Autenticação** | [[Autenticação]] |
| **Threat Model** | [[Threat Model]] |

## Status Geral

> [!todo] Próximos Passos
> - [ ] Revisar e aprovar requisitos
> - [ ] Definir schema Prisma completo
> - [ ] Especificar protótipos de UI
> - [ ] Implementar PoC de criptografia no cliente
> - [ ] Configurar infra Docker Compose
