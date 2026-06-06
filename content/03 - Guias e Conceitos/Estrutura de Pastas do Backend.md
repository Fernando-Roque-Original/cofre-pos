---
title: Estrutura de Pastas do Backend
tags: [arquitetura, estrutura, backend, fastapi, conceitos, grupo-5]
---

# Estrutura de Pastas do Backend

O que cada pasta/arquivo do `back/` faz e por que existe. Serve pra você se localizar no projeto e explicar a organização.

> **A grande ideia:** cada pasta tem **UMA responsabilidade**. Isso se chama *separação de responsabilidades* (separation of concerns). Mantém o código organizado e fácil de mexer — se você precisa mudar uma tabela, vai em `models/`; uma validação, em `schemas/`; uma rota, em `routes/`.

---

## A árvore do projeto

```
poswebreactdeploy/
├── src/                    ← FRONTEND (React + Vite) — não é nosso foco
├── back/                   ← BACKEND (FastAPI) — todo o nosso trabalho
│   ├── main.py             ← ponto de entrada: monta o app, CORS, liga as rotas
│   ├── db/
│   │   └── database.py     ← conexão com o banco (engine, SessionLocal, Base, get_db)
│   ├── models/             ← as TABELAS do banco (SQLAlchemy)
│   │   ├── user_model.py
│   │   ├── aluno_model.py
│   │   └── ... (docente, disciplina, turma, matricula)
│   ├── schemas/            ← os CONTRATOS da API (Pydantic)
│   │   ├── auth_schema.py
│   │   └── aluno_schema.py
│   ├── routes/             ← as ROTAS / endpoints da API
│   │   ├── auth_routes.py
│   │   └── aluno_routes.py
│   ├── core/               ← lógica COMPARTILHADA (segurança: hash + JWT)
│   │   └── security.py
│   ├── alembic/            ← MIGRATIONS (histórico de mudanças do banco)
│   │   ├── env.py
│   │   └── versions/       ← cada arquivo = uma mudança versionada
│   ├── alembic.ini         ← configuração do Alembic
│   ├── seed_usuario.py     ← script que cria um usuário de teste
│   └── create_tables.py    ← (aposentado — agora quem manda é o Alembic)
├── requirements.txt        ← dependências Python
├── .env                    ← segredos (DATABASE_URL) — NÃO vai pro Git
├── .env.example            ← modelo do .env (esse vai pro Git)
└── package.json            ← dependências do React
```

---

## Pasta por pasta (o que é e por que existe)

### `main.py` — o ponto de montagem
O **coração** que liga tudo. Ele:
- cria a aplicação (`app = FastAPI()`)
- configura o CORS
- **registra** as rotas (`app.include_router(...)`)

**Analogia:** é a **recepção** do prédio — quem organiza e aponta cada visitante pra sala certa.

### `db/` — a conexão com o banco
Um arquivo só: `database.py`. Define:
- `engine` → a "tomada" que conecta no PostgreSQL
- `SessionLocal` → cria sessões (conversas) com o banco
- `Base` → a base de onde todos os models herdam
- `get_db()` → entrega uma sessão pras rotas e fecha depois

**Por que separado:** todo mundo (models, rotas, alembic) usa essa conexão. Fica num lugar só.

### `models/` — as tabelas do banco
Cada arquivo é uma **tabela** (em SQLAlchemy). `user_model.py` = tabela `usuarios`, `aluno_model.py` = tabela `alunos`, etc.

**Responsabilidade:** definir **como os dados são guardados** (colunas, tipos, chaves estrangeiras).

**Analogia:** a planta da **estrutura física** do arquivo morto — quais gavetas existem e o que cabe em cada uma.

### `schemas/` — os contratos da API
Cada arquivo define o que **entra** e **sai** da API (em Pydantic). Ex: `aluno_schema.py` tem `AlunoCreate` (o que o cliente manda) e `AlunoResponse` (o que a API devolve).

**Responsabilidade:** **validar** a entrada e **formatar** a saída. É o "porteiro" que confere se o pedido faz sentido.

> **Models ≠ Schemas (confunde muita gente):**
> - **Model** = como o dado é **guardado** no banco (inclui senha_hash, por ex.)
> - **Schema** = como o dado **entra/sai** da API (nunca devolve senha)
> São coisas diferentes de propósito.

### `routes/` — as rotas (endpoints) da API
Cada arquivo é um grupo de **endpoints** (as URLs que o cliente chama). `auth_routes.py` = `/login`, `aluno_routes.py` = `/alunos`.

**Responsabilidade:** receber a requisição, orquestrar (validar → mexer no banco → responder). É a **porta de entrada** de cada funcionalidade.

**Analogia:** as **portas** do prédio, cada uma com um número (a URL). É AQUI que o cliente bate.

### `core/` — lógica compartilhada
Coisas usadas em vários lugares. Hoje tem `security.py` (hash de senha + geração de JWT).

**Por que existe:** o hash é usado no login E no cadastro. Em vez de repetir, fica num lugar só e todo mundo importa.

### `alembic/` — o histórico do banco
As **migrations** — cada mudança na estrutura do banco vira um arquivo versionado em `versions/`. O `env.py` é a configuração que liga o Alembic aos nossos models.

**Responsabilidade:** versionar e aplicar mudanças no banco (criar tabela, adicionar coluna), com possibilidade de **voltar atrás**.

**Analogia:** o **"Git" do banco de dados**.

### Arquivos soltos
- `requirements.txt` → lista de bibliotecas Python (`pip install -r`)
- `.env` → segredos (senha do banco) — **nunca** vai pro Git
- `.env.example` → modelo do `.env`, sem senha real (esse vai pro Git)
- `seed_usuario.py` → cria um usuário de teste
- `create_tables.py` → **aposentado** (o Alembic assumiu)

---

## Como as pastas trabalham JUNTAS (o fluxo de uma requisição)

Quando alguém faz `POST /alunos`:

```
1. routes/aluno_routes.py     recebe a requisição
        ↓
2. schemas/aluno_schema.py    valida os dados (AlunoCreate)
        ↓
3. core/security.py           faz o hash da senha
        ↓
4. models/ + db/              cria o Usuario e o Aluno no banco
        ↓
5. schemas/aluno_schema.py    formata a resposta (AlunoResponse, sem senha)
        ↓
6. routes/                    devolve pro cliente
```

Repara: **cada pasta entra na sua vez, fazendo só a sua parte.** É isso que deixa o código organizado.

---

## Resumo em uma tabela

| Pasta/arquivo | Responsabilidade | Analogia |
|---|---|---|
| `main.py` | monta o app + liga as rotas | recepção do prédio |
| `db/database.py` | conexão com o banco | a tomada |
| `models/` | as tabelas (como guarda) | planta do arquivo morto |
| `schemas/` | contratos (entra/sai) | o porteiro |
| `routes/` | endpoints (as URLs) | as portas |
| `core/` | lógica compartilhada | a caixa de ferramentas |
| `alembic/` | migrations (mudanças do banco) | o Git do banco |

---

## Links

- [[Entendendo FastAPI e CORS]] — o que é FastAPI
- [[Modelagem do Banco de Dados]] — detalhe dos models
- [[Conceitos Python do Código (def, return, etc)]] — os termos do código
- [[RF001 - Registro de implementação (o que fizemos)]] — exemplo real usando essas pastas
