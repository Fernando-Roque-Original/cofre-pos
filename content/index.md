---
title: Índice do Cofre
tags: [índice, moc, home]
---

# Índice do Cofre — Projeto Pós-Graduação (FastAPI)

Porta de entrada do vault. Tudo organizado por pasta. Clica nos links pra navegar.

> **Projeto:** Sistema da Pós-Graduação em Sistemas Web Back-end (IFBA) — Grupo 5
> **Repositório:** https://github.com/hiag-code/poswebreactdeploy (pasta `back/` = FastAPI)

---

## 00 - Comece Aqui
Pra quem está chegando agora ou precisa do contexto geral.
- [[00 - LEIA PRIMEIRO - Repositórios novos do professor]] — a troca de repos, stack, time
- [[Plano de Implementação - Desafio 5]] — visão geral das 13 semanas

## 01 - Planejamento
O que fazer e como dividir.
- [[Como começar - Sprint 1]] — setup do ambiente + divisão das tarefas
- [[Semana 2 - Levantamento de Requisitos]] — requisitos do sistema
- [[Estrutura Acadêmica da Pós-Graduação]] — como a pós funciona (entidades, fluxos, regras)

## 02 - Implementação
O que já foi construído (com explicação).
- [[Modelagem do Banco de Dados]] — as 6 tabelas, diagrama, models
- [[RF001 - Login com JWT]] — o guia passo a passo do login
- [[RF001 - Registro de implementação (o que fizemos)]] — o que mudamos no login e por quê
- [[RF002 - Cadastro de Aluno (guia)]] — o guia do cadastro de aluno
- [[RF002 - Registro de implementação (o que fizemos)]] — o que mudamos no RF002 e por quê
- [[RF003 - Cadastro de Docente (guia)]] — o guia do cadastro de docente
- [[RF003 - Registro de implementação (o que fizemos)]] — o que fizemos no RF003 + aprendizados
- [[Proteger Rotas com JWT (guia)]] — Sprint 2: o porteiro que exige o token (segurança)
- [[Proteger Rotas - Registro de implementação (o que fizemos)]] — o que usamos, como funciona, por que cada coisa fica onde
- [[RF004 - Cadastro de Disciplina (guia)]] — Sprint 2: rota protegida (só admin) pra cadastrar matérias
- [[RF004 - Registro de implementação (o que fizemos)]] — o que usamos, como funciona, divergências e aprendizados

## 03 - Guias e Conceitos
Pra entender e explicar o código.
- [[Entendendo FastAPI e CORS]] — o que é FastAPI e CORS
- [[Tecnologias e Bibliotecas do Projeto]] — FastAPI, SQLAlchemy, pgAdmin, libs... o que cada uma faz
- [[Estrutura de Pastas do Backend]] — o que cada pasta (routes, models, schemas...) faz
- [[Conceitos Python do Código (def, return, etc)]] — cada termo do código explicado
- [[Guia Git - comandos e fluxo seguro]] — clone/branch/push sem bagunçar a main
- [[Quartz - publicar e atualizar o cofre como site]] — transformar o cofre num site pro grupo (e como atualizar)

## Documentos Fonte
Os arquivos originais do desafio (na pasta `Documentos Fonte/`):
- Cronograma - grupo_5.xlsx
- Descrição do Desafio 5.docx
- TCC_Bem_Hur.pdf · TCC_Leticia.pdf

---

## Status do projeto

| Etapa | Status |
|---|---|
| Modelagem do banco (6 tabelas) | ✅ feito |
| Endpoint de Aluno (versão básica) | ✅ feito |
| RF001 — Login com JWT | ✅ feito (5/5) |
| Alembic (migrations) | ✅ no controle do banco |
| RF002 — Cadastro de Aluno (completo) | ✅ feito (4/4) |
| RF003 — Cadastro de Docente | ✅ feito (3/3) |
| Proteger Rotas Privadas (JWT) | ✅ feito (4/4) |
| RF004 — Cadastro de Disciplina | ✅ feito (4/4) |
| RF005 / RF006: turma, matrícula | ⏳ depois |
| Site do cofre (Quartz) pro grupo | ⏳ local pronto, falta publicar |

---

> **Dica:** dá pra deixar essa nota como página inicial do Obsidian em **Configurações → Página inicial** (ou plugin "Homepage").
