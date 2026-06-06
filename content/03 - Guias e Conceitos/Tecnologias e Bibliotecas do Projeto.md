---
title: Tecnologias e Bibliotecas do Projeto
tags: [tecnologias, bibliotecas, stack, conceitos, backend, grupo-5]
---

# Tecnologias e Bibliotecas do Projeto

O que cada tecnologia e biblioteca faz e por que está sendo usada no projeto. Serve pra entender o stack e explicar pro grupo/professor.

## Visão geral do stack

| Camada | Tecnologia | Papel em uma frase |
|---|---|---|
| Linguagem | Python | a linguagem do backend |
| Framework | FastAPI | cria a API (as rotas) |
| Servidor | Uvicorn | roda a aplicação FastAPI |
| Banco | PostgreSQL | onde os dados ficam guardados |
| Driver | psycopg2 | liga o Python ao PostgreSQL |
| ORM | SQLAlchemy | conversa com o banco usando objetos Python |
| Migrations | Alembic | versiona as mudanças do banco |
| GUI do banco | pgAdmin | visualizar e gerenciar o PostgreSQL |
| Validação | Pydantic | valida o que entra e sai da API |
| Senha | passlib + bcrypt | faz o hash das senhas |
| Token | python-jose | gera e valida o JWT |
| Config | python-dotenv | lê o arquivo .env |
| Testes | pytest | testes automatizados |
| Versionamento | Git / GitHub | controle de versão do código |
| Frontend | React + Vite | a interface (telas) |

---

## Linguagem e framework

### Python
A **linguagem** em que o backend é escrito. Foi escolhida porque o desafio pede FastAPI (que é Python) e porque é simples de ler e tem bibliotecas prontas pra tudo.

### FastAPI
O **framework web** que cria a API. Ele recebe as requisições HTTP, organiza as rotas, valida os dados e gera a documentação automática.
- **Por que:** exigido pelo desafio; é rápido, moderno e gera o Swagger sozinho.
- **No nosso código:** `main.py` (`app = FastAPI()`) e tudo em `routes/`.

### Uvicorn
O **servidor** que efetivamente roda a aplicação FastAPI. O FastAPI escreve a lógica; o Uvicorn é quem "liga" e fica escutando as requisições.
- **No nosso código:** o comando `uvicorn main:app --reload`.

---

## Banco de dados

### PostgreSQL
O **banco de dados** relacional onde os dados ficam guardados (tabelas, linhas, colunas). É robusto, gratuito e o padrão da indústria.
- **No nosso código:** o banco `agents_db` rodando no localhost.

### psycopg2
O **driver** do PostgreSQL pra Python. É a "ponte" que permite o Python conversar com o Postgres. Sem ele, o SQLAlchemy não consegue se conectar.
- **No nosso código:** instalado via `psycopg2-binary` no requirements; usado por baixo dos panos.

### SQLAlchemy
O **ORM** (Object-Relational Mapper). Ele deixa a gente mexer no banco usando **objetos Python** em vez de escrever SQL na mão. Por exemplo, `db.query(Aluno).all()` em vez de `SELECT * FROM alunos`.
- **Por que:** escreve menos SQL, menos erro, código mais limpo.
- **No nosso código:** `db/database.py` (engine, Base) e todos os arquivos em `models/`.

### Alembic
A ferramenta de **migrations** — versiona as mudanças na estrutura do banco (criar tabela, adicionar coluna). É como um "Git do banco": cada mudança vira um arquivo, em ordem, com possibilidade de voltar atrás.
- **Por que:** o SQLAlchemy cria tabelas, mas não sabe **alterar** uma que já existe sem perder dados. O Alembic resolve isso.
- **No nosso código:** a pasta `alembic/` e os comandos `alembic revision` / `alembic upgrade`.

### pgAdmin
A **interface gráfica** (GUI) pra ver e gerenciar o PostgreSQL. Em vez de digitar SQL no terminal, você navega visualmente pelas tabelas, vê os dados, cria bancos.
- **Por que:** facilita visualizar o que está no banco (ex: conferir se a tabela `alunos` recebeu as colunas novas).
- **Observação:** o pgAdmin é uma **ferramenta**, não uma biblioteca do código. Vem junto quando você instala o PostgreSQL.

---

## Validação e segurança

### Pydantic
A biblioteca de **validação de dados**. É ela que o FastAPI usa pra conferir se o que chega na API tem o formato certo (ex: email é texto, data_nascimento é uma data válida).
- **No nosso código:** todos os arquivos em `schemas/` (as classes que herdam de `BaseModel`).

### passlib + bcrypt
Fazem o **hash das senhas**. O `bcrypt` é o algoritmo de hash; o `passlib` é uma camada por cima que facilita usar. Juntos garantem que a senha nunca é guardada em texto puro.
- **Por que:** se o banco vazar, as senhas estão em hash, não expostas.
- **No nosso código:** `core/security.py` (`hash_password`, `verify_password`).
- **Observação:** o `bcrypt` está fixado na versão `4.0.1` porque versões mais novas quebram com o `passlib`.

### python-jose
Gera e valida o **JWT** (o token de login). O "jose" vem de "JavaScript Object Signing and Encryption" — é o padrão de tokens assinados.
- **No nosso código:** `core/security.py` (`jwt.encode`).

---

## Utilidades

### python-dotenv
Lê o arquivo **.env** e carrega as variáveis (como a `DATABASE_URL`) pro programa. Mantém os segredos fora do código.
- **No nosso código:** `load_dotenv()` no `db/database.py`.

### pytest
A ferramenta de **testes automatizados**. Permite escrever testes que rodam sozinhos e conferem se a API se comporta como esperado.
- **No nosso código:** ainda não usamos, mas está no requirements pronto pra quando formos testar de forma automatizada.

---

## Ferramentas (fora do código)

### Git e GitHub
O **controle de versão**. O Git guarda o histórico do código no seu PC; o GitHub é onde esse histórico fica online e o grupo compartilha. Ver [[Guia Git - comandos e fluxo seguro]].

### Swagger / OpenAPI
A **documentação interativa** que o FastAPI gera sozinho, em `/docs`. Lista todos os endpoints e deixa testar a API pelo navegador. Ver [[Entendendo FastAPI e CORS]].

### VS Code
O **editor de código** onde a gente escreve tudo.

---

## Frontend (contexto)

### React + Vite
O **React** é a biblioteca que monta as telas (a interface que o usuário vê). O **Vite** é a ferramenta que roda e empacota esse frontend. Não é nosso foco (a gente faz o backend), mas faz parte do projeto e vai consumir a nossa API.
- **No projeto:** a pasta `src/`.

---

## Onde cada uma aparece no nosso código (resumo)

| Arquivo/comando | Tecnologias envolvidas |
|---|---|
| `main.py` | FastAPI |
| `db/database.py` | SQLAlchemy, psycopg2, python-dotenv |
| `models/` | SQLAlchemy |
| `schemas/` | Pydantic |
| `routes/` | FastAPI |
| `core/security.py` | passlib, bcrypt, python-jose |
| `alembic/` | Alembic, SQLAlchemy |
| `uvicorn main:app` | Uvicorn |
| `/docs` | Swagger (gerado pelo FastAPI) |

---

## Links

- [[Estrutura de Pastas do Backend]] - onde cada tecnologia atua
- [[Entendendo FastAPI e CORS]] - detalhe do FastAPI
- [[Modelagem do Banco de Dados]] - SQLAlchemy na prática
- [[Conceitos Python do Código (def, return, etc)]] - os termos do código
