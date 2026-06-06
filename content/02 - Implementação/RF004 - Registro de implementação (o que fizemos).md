---
title: RF004 - Registro de implementação (o que fizemos)
tags: [rf004, disciplina, fastapi, autorizacao, jwt, registro, sprint-2, backend]
status: concluído
sprint: 2
card_trello: "[RF004] Cadastro de Disciplina (POST /disciplinas)"
---

# RF004 - Registro de implementação (o que fizemos)

> Status: concluído (4/4 do checklist).
> Testado: 401 sem token · 403 não-admin · 201 com admin · 409 código duplicado · 422 dados inválidos.

Registro do que foi feito no RF004: o que usamos, como funciona, por que cada coisa fica onde, e os perrengues que resolvemos no caminho.

## O que o card pedia

1. Criar schema Pydantic validando entrada (Nome, Código, Carga Horária, Ementa)
2. Proteger a rota com JWT (apenas secretaria/admin)
3. Validar se o "Código" já existe (HTTP 409)
4. Salvar e retornar HTTP 201

## O que construímos (e por que cada coisa fica onde)

| Arquivo | O que mudou | Por que aqui |
|---|---|---|
| `models/disciplina_model.py` | + coluna `codigo` (única) | é a estrutura da tabela; o `unique=True` é a rede de segurança do 409 |
| `alembic/versions/...add_codigo...` | migration nova | o Alembic versiona a mudança do banco; encadeada **após** a do RF003 |
| `schemas/disciplina_schema.py` | `DisciplinaCreate` / `DisciplinaResponse` | o contrato de entrada/saída; todos os schemas ficam em `schemas/` |
| `core/security.py` | + `require_admin` | lógica de autorização, **reaproveitável**; mesmo lugar do `get_current_user` |
| `routes/disciplina_routes.py` | `POST /disciplinas` | o endpoint; rotas ficam em `routes/` |
| `main.py` | registra o router | sem registrar, a rota existe no arquivo mas o app não a conhece (404) |

## Como funciona (o fluxo)

```
Cliente: POST /disciplinas  + Authorization: Bearer <token>
   ↓ Depends(require_admin)
   │    → get_current_user: o token vale? (senão -> 401)
   │    → require_admin: a role é admin? (senão -> 403)
   ↓ FastAPI valida o corpo com DisciplinaCreate (senão -> 422)
   ↓ checa codigo duplicado no banco (se já existe -> 409)
   ↓ salva: add → commit → refresh (pega o id gerado)
   ↓ devolve 201 + DisciplinaResponse (com o id)
```

## Conceito novo: autorização (e a diferença middleware x dependency)

Até o RF "Proteger Rotas" a gente só tinha **autenticação** ("quem é você?"). O RF004 trouxe **autorização** ("você PODE fazer isso?").

| | Pergunta | Quem faz | Erro |
|---|---|---|---|
| Autenticação | Tá logado? (token válido) | `get_current_user` | 401 |
| Autorização | É admin? (role) | `require_admin` | 403 |

> [!important] Middleware x Dependency (o termo do card)
> O card dizia "aplicar o middleware de JWT", mas o que usamos é uma **dependency** (`Depends`), não um middleware.
> - **Middleware** = roda em **todas** as rotas (global). No projeto, o único middleware de verdade é o **CORS** (`app.add_middleware`).
> - **Dependency** = roda **só** nas rotas onde você põe o `Depends`. É o jeito do FastAPI pra proteger **rotas específicas**.
>
> Usamos dependency porque queremos proteger só algumas rotas (criar disciplina) e deixar outras públicas (o login **precisa** ser público, senão ninguém pega o token). O objetivo do card (proteger a rota) foi cumprido — só com o mecanismo certo.

## As 2 divergências do card que resolvemos

> [!check] 1. O model não tinha "Código"
> O `disciplina_model.py` tinha nome, ementa, carga_horaria, link — **sem `codigo`**. Como o card pede o campo e ainda checa duplicado por ele, **adicionamos a coluna `codigo` (única)** + migration. Disciplina de verdade tem código mesmo (tipo "BD101").

> [!check] 2. Não existe papel "secretaria"
> Os papéis são estudante / docente / admin. O card diz "secretaria/admin"; como **a secretaria usa contas com role `admin`**, a regra `require_admin` já cobre os dois.

## O perrengue da cadeia de migrations (e como resolvemos)

Ao gerar a migration do `codigo`, deu erro **"Can't locate revision 8bea859b015e"**. Motivo: o banco já estava na migration do RF003 (`8bea...`), mas a branch do RF004 tinha nascido da `main` — que ainda não tinha as migrations do RF002/RF003 (os PRs não tinham mergeado).

Como resolvemos:
1. Mergeamos RF002 → RF003 na `main` (em ordem — migration é uma corrente).
2. Resolvemos o conflito do `main.py` no merge do RF003 mantendo **todas** as rotas (`docente_routes` + `me_routes`).
3. Recriamos a branch do RF004 a partir do RF003 (que tinha a cadeia completa que bate com o banco).

> [!tip] Lição
> Branch com migration tem que **nascer da base que tem as migrations anteriores**. E PRs com migration se mergeiam **na ordem** (RF002 antes do RF003 antes do RF004).

## Os 4 itens do checklist -> onde ficam

| # | Item | Onde |
|---|---|---|
| 1 | Schema validando entrada | `disciplina_schema.py` (`Field(min_length=2)`, `Field(gt=0)`) |
| 2 | Proteção JWT (só admin) | `require_admin` + `Depends` na rota |
| 3 | Código duplicado -> 409 | `db.query(...).filter(codigo)...` na rota |
| 4 | Salvar -> 201 | `add/commit/refresh` + `status_code=201` |

## Aprendizados

- **Autorização por papel:** `require_admin` depende do `get_current_user` (a corrente: token vale? → é admin?). 401 = não logado; 403 = logado sem permissão.
- **Middleware x dependency:** proteção de rota específica é **dependency**, não middleware.
- **Cadeia de migrations:** a base da branch importa; mergear PRs com migration em ordem.
- **Indentação em Python = escopo:** o `require_admin` chegou a ficar "dentro" do `get_current_user` por causa da indentação. `def` na margem = função do arquivo; `def` indentado = função aninhada. Corpo sempre 4 espaços pra dentro.

## Links

- [[RF004 - Cadastro de Disciplina (guia)]] - o guia/roteiro
- [[Proteger Rotas - Registro de implementação (o que fizemos)]] - o `get_current_user` que reusamos
- [[RF002 - Registro de implementação (o que fizemos)]] - o padrão de schema + rota + 409
- [[Modelagem do Banco de Dados]] - a tabela disciplinas
- Card: [RF004] Cadastro de Disciplina (POST /disciplinas)
