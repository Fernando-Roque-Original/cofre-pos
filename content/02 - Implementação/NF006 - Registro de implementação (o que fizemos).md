---
title: NF006 - Registro de implementação (o que fizemos)
tags: [nf006, middleware, auditoria, logs, seguranca, registro, sprint-3, backend]
status: concluído
sprint: 3
card_trello: "[NF006] Middleware de Logs de Auditoria"
---

# NF006 - Registro de implementação (o que fizemos)

> Status: concluído (3/3).
> Testado: POST de admin loga (usuário 1 + 201) · POST negado loga (nulo + 401) · GET não loga.

Registro do que foi feito na NF006: o middleware de auditoria, como funciona, e o perrengue da migration vazia (com uma lição importante do Alembic).

## O que o card pedia

1. Criar um **middleware** no FastAPI pra interceptar requisições em rotas administrativas.
2. Extrair **IP, rota, ID do usuário (do JWT) e Data/Hora** da ação.
3. Salvar numa **tabela de Logs** no banco.

## O que construímos (e por que cada coisa fica onde)

| Arquivo | O que | Por que |
|---|---|---|
| `models/log_model.py` | tabela `logs` (ip, rota, metodo, usuario_id, status_code, data_hora) | é a estrutura do registro |
| `main.py` | o middleware `@app.middleware("http")` + `import models.log_model` | o middleware precisa do `app`, então mora no main |
| `alembic/env.py` | **+ `import models.log_model`** (correção) | senão o autogenerate não detecta a tabela nova |
| `alembic/versions/...logs` | migration da tabela | versiona a criação da tabela |

## Como funciona (o fluxo)

```
Requisição chega
   ↓ @app.middleware("http")
   ↓ response = await call_next(request)   → deixa a rota rodar (e pega o status_code)
   ↓ é POST/PUT/PATCH/DELETE?  (GET nao audita)
   ↓ le o header Authorization, decodifica o JWT na mao → usuario_id (ou None)
   ↓ SessionLocal() → salva Log(ip, rota, metodo, usuario_id, status_code)
   ↓ return response
```

## Conceito central: middleware DE VERDADE (x dependency)

| | Roda quando | Exemplo |
|---|---|---|
| **Dependency** (`Depends`) | só nas rotas marcadas | require_admin |
| **Middleware** | em **toda** requisição | CORS, e agora a **auditoria** |

Três coisas que só existem no middleware (não dá pra usar `Depends`):
- **Decodificar o JWT na mão** — o middleware roda antes das dependencies.
- **Criar a sessão na mão** (`SessionLocal()`) — o `get_db` é dependency.
- **Registrar depois** (`await call_next`) — pra ter o `status_code` da resposta.

## O perrengue: a migration ficou VAZIA (a lição do env.py)

O primeiro `alembic revision --autogenerate` gerou uma migration **sem** o `create_table` — e ao aplicar, a tabela `logs` **não foi criada**. Resultado: o middleware tentava gravar numa tabela inexistente e **todo POST dava 500**.

**Causa:** o `alembic/env.py` importa os models **um por um**, e faltava o `import models.log_model`. Sem esse import, o autogenerate não "enxerga" a tabela.

**Correção:** adicionar o import no `env.py`, desfazer a migration vazia, e regenerar (aí veio o `create_table` de verdade).

> [!important] Lição
> Quando criar um **model novo**, importa ele **no `alembic/env.py` também** — não só no `main.py`. O Alembic roda o env.py dele pra saber quais tabelas existem. (Já consertei o env.py, então dos próximos models ele detecta sozinho.)

## O resultado do teste

```
[id, ip, rota, metodo, usuario_id, status]
(3, '127.0.0.1', '/disciplinas/', 'POST', None, 401)  ← acao NEGADA (sem token)
(2, '127.0.0.1', '/disciplinas/', 'POST', '1',  201)  ← admin criou (quem: usuario 1)
(1, '127.0.0.1', '/login',        'POST', None, 200)  ← login
```

Auditou o sucesso, o **login**, e até a **tentativa negada** (com o status) — e ignorou o GET.

## Os 3 itens do checklist -> onde ficam

| # | Item | Onde |
|---|---|---|
| 1 | Middleware que intercepta | `@app.middleware("http")` no `main.py` |
| 2 | IP, rota, usuário (JWT), hora | dentro do middleware (`request.client.host`, `request.url.path`, `jwt.decode`, `func.now()`) |
| 3 | Salvar na tabela de Logs | `log_model.py` + `db.add(Log(...))` |

## Aprendizados

- **Middleware x dependency** virou prática: middleware pra "toda requisição", dependency pra "rota específica".
- **JWT e sessão na mão** no middleware (não tem `Depends`).
- **Registrar depois do `call_next`** pra ter o `status_code` (audita até o que foi negado).
- **`env.py` do Alembic precisa importar todo model novo** — foi o bug da migration vazia.

## Links

- [[NF006 - Middleware de Logs de Auditoria (guia)]] - o guia/roteiro
- [[RF004 - Registro de implementação (o que fizemos)]] - middleware x dependency
- [[Proteger Rotas - Registro de implementação (o que fizemos)]] - o JWT decodificado aqui
- Card: [NF006] Middleware de Logs de Auditoria
