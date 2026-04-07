---
title: "Requisitos Funcionais"
tags:
  - fatesck
  - requisitos
  - funcionais
---

# Requisitos Funcionais

## Autenticação

| ID | Requisito | Descrição |
|----|-----------|-----------|
| **RF001** | Autenticação | Registro com e-mail/senha + OAuth (Google, Microsoft) |
| **RF001.1** | JWT + Refresh | Login com `access_token` (JWT) + `refresh_token` (opaco) |
| **RF001.2** | RBAC | Controle de acesso: `admin`, `professor`, `aluno` |
| **RF001.3** | Recuperação | Fluxo seguro com invalidação de sessão |

> [!info] Implementação
> Detalhes do fluxo de autenticação em [[Autenticação]].

## Gestão de Turmas

| ID | Requisito | Descrição |
|----|-----------|-----------|
| **RF002** | Criar turmas | Nome, descrição, código de acesso, período letivo |
| **RF002.1** | Ingressar | Via código de convite ou link direto |
| **RF002.2** | Gerenciar membros | Adicionar/remover alunos manualmente |
| **RF002.3** | Listar | Turmas ativas e arquivadas por perfil |

## Gestão de Atividades

| ID | Requisito | Descrição |
|----|-----------|-----------|
| **RF003** | Criar atividades | Título, descrição, prazo, peso, anexos |
| **RF003.1** | Visualizar | Atividades pendentes e históricas |
| **RF003.2** | Submeter | Anexos (PDF, DOCX, imagens, ZIP) |
| **RF003.3** | Avaliar | Nota + feedback escrito |
| **RF003.4** | Timestamps | Registro de entrega e atraso |

## Chat E2E

| ID | Requisito | Descrição |
|----|-----------|-----------|
| **RF004** | Mensagens 1:1 | Entre membros de uma mesma turma |
| **RF004.1** | Mensagens em grupo | Chat de turma ou grupo |
| **RF004.2** | Anexos criptografados | Imagens, documentos, áudios |
| **RF004.3** | Status | Enviado, entregue, lido |
| **RF004.4** | E2E obrigatório | Descriptografia apenas no cliente |

## Dashboards

| ID | Requisito | Descrição |
|----|-----------|-----------|
| **RF005** | Painel do Professor | Visão de turmas, estatísticas, gráficos, exportação |
| **RF006** | Painel do Aluno | Pendências, prazos, notas, calendário, notificações |

## Notificações

| ID | Requisito | Descrição |
|----|-----------|-----------|
| **RF007** | Push (web/mobile) | Eventos relevantes para o usuário |
| **RF007.1** | Preferências | Configuração por usuário |
| **RF007.2** | Sem conteúdo | Nunca transporta dados criptografados — só metadados |

## Arquivos e Administração

| ID | Requisito | Descrição |
|----|-----------|-----------|
| **RF008** | Gerenciamento de Arquivos | Upload/download com criptografia no cliente, armazenamento em object storage |
| **RF009** | Administração | CRUD de usuários, métricas, logs de auditoria |

> [!example] Entidades Relacionadas
> - Usuários: [[Entidade - Users]]
> - Turmas: [[Entidade - Classes]]
> - Atividades: [[Entidade - Assignments]]
> - Submissões: [[Entidade - Submissions]]
