---
title: Plano de Implementação — Desafio 5
tags: [fastapi, pos-graduacao, ifba, backend, planejamento]
created: 2026-05-20
status: em-andamento
grupo: 5
---

# Plano de Implementação — Sistema de Pós-Graduação (FastAPI)

> **Projeto:** Sistema da Pós-Graduação em Sistemas Web Back-end utilizando FastAPI (IFBA)
> **Prazo total:** 04/05/2026 → 03/08/2026 (13 semanas)

> [!important] Atualização — repositórios novos do professor
> O professor trocou os repos. Os antigos (`crescenciolima/pos-web`, `hiag-code/pos-web`) saíram. Agora:
> - Backend **referência** (Node + JSON, sem login): https://github.com/DjanInfo/poswebserver
> - Frontend (React/Vite): https://github.com/DjanInfo/poswebreactdeploy
>
> A gente continua em **FastAPI**; o repo do professor é o **contrato da API** (quais rotas e dados existem). Detalhes em [[00 - LEIA PRIMEIRO - Repositórios novos do professor]]. As seções abaixo que falam de Firebase descrevem o sistema **antigo** dos TCCs — servem de contexto histórico, mas a referência atual é o backend Node do professor.

---

## Contexto e ponto de partida

### O que já existe (TCCs anteriores)

Os TCCs de **Bem-Hur Ganem** e **Letícia Porto** construíram um sistema de **processo seletivo** para o PGDW/IFBA usando:

| Item | Tecnologia |
|---|---|
| Frontend | Next.js (TypeScript) + SSG/SSR |
| Banco de dados | Firebase Firestore (NoSQL) |
| Armazenamento | Firebase Cloud Storage |
| Autenticação | Firebase Authentication + JWT |
| Estilização | Bootstrap 4 + FontAwesome |
| Deploy | Vercel (https://pos-web-ifba.vercel.app/) |

**O que esses sistemas fazem:**
- Área pública: página do curso, professores, notícias, TCCs publicados
- Área administrativa: gerenciar editais, resultados, usuários (Master/Administrador)
- Área do aluno: inscrição no processo seletivo, upload de documentos, acompanhamento de resultado

### O que o Desafio 5 pede (novo)

Construir o **back-end acadêmico** (pós-admissão) com FastAPI, substituindo o Firebase como camada de dados e adicionando gestão acadêmica completa. O frontend pode eventualmente migrar de Firebase → FastAPI.

---

## Arquitetura proposta

```
┌─────────────────────────────────────────────────────┐
│                  Frontend (Next.js)                  │
│           https://pos-web-ifba.vercel.app            │
└──────────────────────┬──────────────────────────────┘
                       │ HTTP/REST (JWT)
┌──────────────────────▼──────────────────────────────┐
│              FastAPI — API Gateway                   │
│  /docs (Swagger)   /redoc   /api/v1/...              │
│                                                      │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌────────┐  │
│  │ Alunos   │ │Professores│ │Disciplinas│ │Turmas  │  │
│  └──────────┘ └──────────┘ └──────────┘ └────────┘  │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌────────┐  │
│  │Matrículas│ │ Histórico│ │  Usuários │ │Relatór.│  │
│  └──────────┘ └──────────┘ └──────────┘ └────────┘  │
└──────────────────────┬──────────────────────────────┘
                       │ SQLAlchemy ORM
┌──────────────────────▼──────────────────────────────┐
│          PostgreSQL (produção) / SQLite (dev)         │
└─────────────────────────────────────────────────────┘
```

### Stack técnico recomendado

| Camada | Tecnologia | Justificativa |
|---|---|---|
| Framework | **FastAPI** | Exigido pelo desafio; Swagger automático |
| ORM | **SQLAlchemy 2.x** + Alembic | Migrations versionadas |
| Banco (dev) | **SQLite** | Zero config, roda localmente |
| Banco (prod) | **PostgreSQL** | Robusto, gratuito no Railway/Supabase |
| Autenticação | **JWT** (python-jose) + bcrypt | Compatível com o frontend existente |
| Validação | **Pydantic v2** | Nativo no FastAPI, docs automáticos |
| Testes | **pytest** + httpx | Testes de integração da API |
| Deploy | **Railway** ou **Render** | Free tier generoso |
| CSV Import | **pandas** ou csv stdlib | Importação em lote de dados |

### Estrutura de pastas do projeto

```
pos-api/
├── app/
│   ├── main.py              # Entrada FastAPI, CORS, routers
│   ├── database.py          # Engine SQLAlchemy, SessionLocal
│   ├── dependencies.py      # get_db(), get_current_user()
│   ├── models/              # SQLAlchemy models (tabelas)
│   │   ├── aluno.py
│   │   ├── professor.py
│   │   ├── disciplina.py
│   │   ├── turma.py
│   │   ├── matricula.py
│   │   └── usuario.py
│   ├── schemas/             # Pydantic schemas (request/response)
│   ├── routers/             # Um arquivo por módulo
│   │   ├── alunos.py
│   │   ├── professores.py
│   │   ├── disciplinas.py
│   │   ├── turmas.py
│   │   ├── matriculas.py
│   │   ├── historico.py
│   │   ├── usuarios.py
│   │   └── relatorios.py
│   ├── services/            # Lógica de negócio
│   └── auth/                # JWT, hashing, dependências de auth
├── alembic/                 # Migrations
├── tests/
├── imports/                 # CSVs de importação
├── requirements.txt
└── .env
```

---

## Roteiro semana a semana (alinhado ao cronograma)

### Mês 1 — Planejamento + Protótipo

#### Semana 1 (04–10/mai) Passada
- [x] Reunião inicial com os grupos
- Entrega: Vídeo da reunião

#### Semana 2 (11–17/mai) — Levantamento de requisitos
- [ ] Mapear entidades: Aluno, Professor, Disciplina, Turma, Matrícula, Histórico, Usuário
- [ ] Definir regras de negócio: vagas por turma, pré-requisitos, notas mínimas
- [ ] Estudar o código do [[pos-web]] (Firebase) para entender modelos existentes
- [ ] Rascunhar diagrama entidade-relacionamento (DER)
- **Entrega:** Documento de Requisitos (`.md` ou `.pdf`)

> **Dica:** Os modelos do Firestore nos TCCs (User, Course, Teacher, News, Work, SelectiveProcess, Subscription) ajudam a entender o domínio — mas o novo sistema é relacional (SQL), então normalize as entidades.

#### Semana 3 (18–24/mai) — Arquitetura e tecnologias
- [ ] Confirmar stack: FastAPI + SQLAlchemy + PostgreSQL
- [ ] Criar repositório Git (`pos-api`)
- [ ] Configurar ambiente: `pyproject.toml` ou `requirements.txt`, `.env.example`
- [ ] Definir padrão de endpoints REST: `/api/v1/alunos`, `/api/v1/turmas/{id}/matriculas`, etc.
- [ ] Esboçar DER final no dbdiagram.io ou draw.io
- **Entrega:** Documento de arquitetura + DER

#### Semana 4 (25–31/mai) — Protótipo
- [ ] Criar projeto FastAPI base: `main.py`, roteadores, CORS
- [ ] Configurar SQLite para dev + Alembic para migrations
- [ ] Implementar modelo de Usuário + autenticação JWT (login/registro)
- [ ] Testar `/docs` (Swagger) funcionando
- [ ] Criar telas no Figma/Canva para referência visual do admin
- **Entrega:** Protótipo inicial (API rodando localmente com Swagger)
- **Reunião 1 (07/jun):** Validação do protótipo com professor

---

### Mês 2 — Desenvolvimento do sistema

#### Semana 5 (01–07/jun) — Banco de dados e modelos
- [ ] Implementar todos os models SQLAlchemy (ver lista abaixo)
- [ ] Criar migrations Alembic para cada model
- [ ] Implementar importação de CSV para Alunos e Professores
- [ ] Criar seeds de dados para desenvolvimento
- **Entrega:** Modelo do banco funcional + migrations

**Models prioritários:**
```python
# Aluno: matrícula, nome, email, CPF, curso, período_ingresso
# Professor: nome, email, CPF, titulação, áreas
# Disciplina: código, nome, carga_horária, pré_requisitos[]
# Turma: disciplina_id, professor_id, semestre, vagas, vagas_disponíveis
# Matrícula: aluno_id, turma_id, data, status, nota_final
# HistoricoAcademico: aluno_id, disciplinas cursadas, situação
# Usuario: email, senha_hash, perfil (admin/professor/aluno), ativo
```

#### Semana 6 (08–14/jun) — Cadastro de Alunos e Professores
- [ ] `GET/POST/PUT/DELETE /api/v1/alunos`
- [ ] `GET/POST/PUT/DELETE /api/v1/professores`
- [ ] Upload CSV para importação em lote
- [ ] Filtros de busca (nome, matrícula, e-mail)
- [ ] Paginação em todos os listagens
- **Entrega:** Endpoints funcionando + testados

#### Semana 7 (15–21/jun) — Cadastro de Disciplinas
- [ ] `GET/POST/PUT/DELETE /api/v1/disciplinas`
- [ ] Controle de pré-requisitos entre disciplinas
- [ ] Validação de carga horária
- **Entrega:** Módulo de disciplinas funcionando

#### Semana 8 (22–28/jun) — Módulo de Turmas
- [ ] `GET/POST/PUT/DELETE /api/v1/turmas`
- [ ] `GET /api/v1/turmas/{id}/alunos` — alunos matriculados
- [ ] Controle de vagas (disponíveis/total)
- [ ] Associação Professor ↔ Turma ↔ Disciplina
- **Entrega:** Módulo de turmas funcionando
- **Reunião 2 (05/jul):** Sistema parcial funcionando

---

### Mês 3 — Finalização + Testes + Entrega

#### Semana 9 (29/jun–05/jul) — Módulo de Matrículas
- [ ] `POST /api/v1/turmas/{id}/matriculas` — matricular aluno
- [ ] `DELETE /api/v1/matriculas/{id}` — cancelar matrícula
- [ ] Validações: vagas disponíveis, pré-requisitos cumpridos, matrícula duplicada
- [ ] Regras de negócio: limite de disciplinas por aluno/semestre
- **Entrega:** Módulo de matrículas funcionando

#### Semana 10 (06–12/jul) — Histórico acadêmico
- [ ] `GET /api/v1/alunos/{id}/historico`
- [ ] Cálculo de IRA (índice de rendimento acadêmico)
- [ ] Situação por disciplina: aprovado/reprovado/trancado
- [ ] Geração de histórico em PDF (opcional: ReportLab)
- **Entrega:** Histórico acadêmico funcionando

#### Semana 11 (13–19/jul) — Testes e correções
- [ ] Testes automatizados com `pytest` + `httpx`
- [ ] Cobrir casos de borda: CPF duplicado, vagas esgotadas, pré-requisito faltando
- [ ] Revisão de segurança: validar permissões por perfil (admin vs aluno vs professor)
- [ ] Rate limiting básico
- **Entrega:** Sistema testado e ajustado

#### Semana 12 (20–26/jul) — Ajustes finais e documentação
- [ ] Revisar e completar Swagger (`/docs`) com exemplos em todos os endpoints
- [ ] Módulo de relatórios:
  - [ ] `GET /api/v1/relatorios/turmas` — ocupação por turma
  - [ ] `GET /api/v1/relatorios/alunos` — situação acadêmica
  - [ ] `GET /api/v1/relatorios/matriculas` — matrículas por semestre
- [ ] Preparar deploy em produção (Railway/Render)
- [ ] Preparar slides de apresentação
- **Entrega:** Sistema completo + documentação

#### Semana 13 (27/jul–03/ago) — Relatório final
- [ ] Escrever relatório técnico: arquitetura, decisões, resultados
- [ ] Gravar vídeo demo da API (Swagger + Postman)
- **Entrega:** Relatório + apresentação
- **Reunião 3 (03/ago):** Entrega final

---

## Pontos de integração com o frontend (pos-web)

O frontend atual usa Firebase diretamente. Para integrar com a nova API FastAPI:

1. **Autenticação:** O frontend usa `Firebase Authentication`. A nova API emite JWT próprio. Estratégia: manter Firebase Auth no frontend por ora, mas validar o token via endpoint `/auth/verify`.
2. **CORS:** Configurar FastAPI para aceitar origem `https://pos-web-ifba.vercel.app`.
3. **Contratos de API:** Manter os campos dos schemas Pydantic compatíveis com os modelos Firestore existentes (ex: mesmo campo `email`, `nome`, etc.).
4. **Migração gradual:** Substituir chamadas Firebase por chamadas à FastAPI módulo a módulo — sem big-bang.

---

## Riscos e mitigações

| Risco | Probabilidade | Mitigação |
|---|---|---|
| Regras de negócio complexas (pré-requisitos) | Alta | Implementar simples primeiro, evoluir |
| Integração Firebase ↔ FastAPI travada | Média | Manter Firebase Auth; só migrar dados depois |
| Deploy em produção com problemas | Média | Usar Railway (free tier fácil) desde a Semana 4 |
| Atraso na Semana 6–8 (núcleo do sistema) | Média | Priorizar CRUD básico; relatórios são opcionais no MVP |
| CSV malformado na importação | Baixa | Validar com pandas antes de inserir; retornar erros detalhados |

---

## Referências úteis

- FastAPI docs: https://fastapi.tiangolo.com/
- SQLAlchemy 2.x: https://docs.sqlalchemy.org/en/20/
- Alembic migrations: https://alembic.sqlalchemy.org/
- JWT com FastAPI: https://fastapi.tiangolo.com/tutorial/security/oauth2-jwt/
- Repositório frontend: https://github.com/crescenciolima/pos-web/tree/main
- Deploy gratuito: https://railway.app / https://render.com

---

## Definição de "pronto" por módulo

- [ ] Endpoint responde status correto (200/201/404/422)
- [ ] Validações de entrada via Pydantic documentadas no Swagger
- [ ] Permissão de perfil verificada
- [ ] Teste automatizado cobrindo happy path + 1 caso de erro
- [ ] Nenhum dado sensível (senha em texto puro) na resposta

---

*Fontes: Cronograma - grupo_5.xlsx, Descrição do Desafio 5.docx, TCC_Bem_Hur.pdf, TCC_Leticia.pdf e repositório pos-web.*
