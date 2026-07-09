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
- [[RF005 - Cadastro de Turma (guia)]] — Sprint 2: rota de turma (admin) + validar FK (404)
- [[RF005 - Registro de implementação (o que fizemos)]] — o que foi feito, a revisão e os aprendizados
- [[RF006 - Efetuar Matrícula (guia)]] — Sprint 2: aluno se matricula; vagas + transação
- [[RF006 - Registro de implementação (o que fizemos)]] — o que usamos, aluno do token, transação
- [[Relatórios e Exportação CSV (Sprint 3, time)]] — os relatórios do time (RF007-009) + o CSV da Ana
- [[NF006 - Middleware de Logs de Auditoria (guia)]] — Sprint 3: o middleware "de verdade" de auditoria
- [[NF006 - Registro de implementação (o que fizemos)]] — o middleware, o fluxo, e a lição do env.py
- [[NF001 - Documentação Swagger (guia)]] — Sprint 3: polir o /docs (títulos, descrições, exemplos)
- [[NF001 - Registro de implementação (o que fizemos)]] — o que polimos no Swagger + a verificação
- [[RF010 e RF011 - Notícias e Editais (guia)]] — Sprint 3: notícias + editais (CRUD gêmeo, GET público / POST admin)
- [[RF010 e RF011 - Registro de implementação (o que fizemos)]] — o que fizemos, as decisões e o teste

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
| RF001 — Login com JWT | ✅ feito (5/5) |
| Alembic (migrations) | ✅ no controle do banco |
| RF002 — Cadastro de Aluno | ✅ feito (4/4) |
| RF003 — Cadastro de Docente | ✅ feito (3/3) |
| Proteger Rotas Privadas (JWT) | ✅ feito (4/4) |
| RF004 — Cadastro de Disciplina | ✅ feito (4/4) |
| RF005 — Cadastro de Turma | ✅ feito (4/4) |
| RF006 — Efetuar Matrícula | ✅ feito (6/6) |
| RF007/008/009 — Relatórios (Hiago) | ✅ feito |
| Exportação CSV (Ana) | ✅ feito |
| NF006 — Logs de Auditoria | ✅ feito (3/3) |
| NF001 — Documentação Swagger | ✅ feito (4/4) |
| RF010 — Modelo de Notícias | ✅ feito |
| RF011 — Modelo de Editais | ✅ feito |
| Site do cofre (Quartz) pro grupo | ✅ publicado |

---

> **Dica:** dá pra deixar essa nota como página inicial do Obsidian em **Configurações → Página inicial** (ou plugin "Homepage").
