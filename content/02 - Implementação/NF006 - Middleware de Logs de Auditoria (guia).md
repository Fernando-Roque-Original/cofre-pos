---
title: NF006 - Middleware de Logs de Auditoria (guia)
tags: [nf006, middleware, auditoria, logs, seguranca, sprint-3, guia]
status: em andamento
sprint: 3
card_trello: "[NF006] Middleware de Logs de Auditoria"
---

# NF006 - Middleware de Logs de Auditoria (guia)

> Status: em andamento (Sprint 3).
> Ferramenta de segurança pra coordenação saber **quem fez o quê** no painel administrativo.

## O que o card pede

1. Criar um **middleware** no FastAPI pra interceptar requisições em rotas administrativas.
2. Extrair o **IP, a rota acessada, o ID do usuário (pelo JWT) e a Data/Hora** da ação.
3. Salvar esse registro numa **tabela de Logs** no banco (ou em arquivo). → **Escolhemos o banco** (dá pra consultar depois).

## Conceito central: agora é um MIDDLEWARE de verdade

Lembra da diferença que a gente viu no RF006?

| | Roda quando | A gente usou em |
|---|---|---|
| **Dependency** (`Depends`) | só nas rotas que você marca | proteção (require_admin) |
| **Middleware** | em **TODA** requisição (global) | CORS — e agora a auditoria |

A NF006 é o caso perfeito de middleware: a gente quer registrar **todas** as ações, sem ter que marcar rota por rota. O middleware "abraça" a requisição — roda um código, deixa a rota acontecer, e roda mais código depois.

## O que vamos construir

```
Passo 1  models/log_model.py   -> tabela `logs` (+ migration)
Passo 2  main.py               -> o middleware @app.middleware("http")
Passo 3  testar                -> fazer uma acao e ver o log aparecer
```

## Passo 1 — a tabela de logs (model + migration)

Cria **`back/models/log_model.py`**:
```python
from sqlalchemy import Column, Integer, String, DateTime
from sqlalchemy.sql import func
from db.database import Base


class Log(Base):
    __tablename__ = "logs"

    id = Column(Integer, primary_key=True, index=True)
    ip = Column(String(50))
    rota = Column(String(255))
    metodo = Column(String(10))
    usuario_id = Column(String(50), nullable=True)   # pode ser nulo (acao sem login)
    status_code = Column(Integer)
    data_hora = Column(DateTime, server_default=func.now())
```
**Por que cada campo:**
- `ip` → de onde veio a ação.
- `rota` → qual endpoint (ex: `/disciplinas`).
- `metodo` → POST / PUT / DELETE (que tipo de ação).
- `usuario_id` → **quem** fez (vem do token). `nullable=True` porque uma ação sem login (ex: um login que falhou) não tem usuário.
- `status_code` → deu certo (201) ou foi barrado (403)? Audita até as tentativas negadas.
- `data_hora` → **quando** (o banco preenche sozinho com `func.now()`).

Depois:
1. Registra o model no `main.py` (junto com os outros imports de model): `import models.log_model`
2. Gera e aplica a migration (em `back/`): `alembic revision --autogenerate -m "add tabela de logs"` + `alembic upgrade head`

## Passo 2 — o middleware (no `main.py`)

Depois da criação do `app`, adiciona:
```python
from fastapi import Request
from jose import jwt, JWTError
from core.security import JWT_SECRET, JWT_ALGORITHM
from db.database import SessionLocal
from models.log_model import Log


@app.middleware("http")
async def log_auditoria(request: Request, call_next):
    # 1) deixa a requisicao rodar e pega a resposta
    response = await call_next(request)

    # 2) so audita acoes que MODIFICAM (POST/PUT/PATCH/DELETE)
    if request.method in ("POST", "PUT", "PATCH", "DELETE"):
        # 3) descobre o usuario pelo token (middleware nao usa Depends)
        usuario_id = None
        auth = request.headers.get("Authorization")
        if auth and auth.startswith("Bearer "):
            try:
                payload = jwt.decode(auth[7:], JWT_SECRET, algorithms=[JWT_ALGORITHM])
                usuario_id = payload.get("sub")
            except JWTError:
                pass

        # 4) salva o log (sessao criada na mao)
        db = SessionLocal()
        try:
            db.add(Log(
                ip=request.client.host,
                rota=request.url.path,
                metodo=request.method,
                usuario_id=usuario_id,
                status_code=response.status_code,
            ))
            db.commit()
        finally:
            db.close()

    return response
```

**Explicando cada parte (é o coração da NF006):**
- `@app.middleware("http")` → registra um middleware que roda em **toda** requisição HTTP.
- `async def log_auditoria(request, call_next)` → o middleware recebe a `request` e um `call_next` (a função que executa a rota de verdade).
- `response = await call_next(request)` → **deixa a requisição rodar** e guarda a resposta. Por isso conseguimos pegar o `status_code` — a gente registra **depois** da ação acontecer.
- `if request.method in (...)` → só registra as ações que **mudam** algo (POST/PUT/PATCH/DELETE). GET (leitura) não é "ação", então não audita (evita encher a tabela).
- **Decodificar o JWT na mão** → o middleware roda **antes** das dependencies, então não dá pra usar o `get_current_user`. A gente lê o header `Authorization` e decodifica com o mesmo `JWT_SECRET`. Sem token/inválido → `usuario_id = None`.
- **`SessionLocal()`** → o `get_db` é uma dependency (não serve aqui). Então criamos a sessão do banco **na mão** e fechamos no `finally`.

> [!note] Por que registrar DEPOIS (com `await call_next`)?
> Se a gente logasse antes, não saberia o resultado. Registrando depois, dá pra guardar o `status_code` — então o log mostra tanto o que deu certo (201) quanto o que foi **negado** (403), que é ouro pra auditoria.

## Passo 3 — testar

1. Sobe o servidor, faz uma ação de admin (ex: `POST /disciplinas`).
2. Confere a tabela: `SELECT * FROM logs;` — tem que ter uma linha com o ip, `rota=/disciplinas`, `metodo=POST`, o `usuario_id` e o `status_code`.
3. Testa uma ação **negada** (POST sem ser admin → 403) → deve logar com `status_code=403` e o `usuario_id` de quem tentou.

> [!tip] Bônus (não é obrigatório no card)
> Pra coordenação **ver** os logs, dá pra adicionar depois um `GET /logs` (só admin) que lista a tabela. O card só pede pra **salvar**, mas o endpoint deixa a ferramenta completa.

## Os 3 itens do checklist -> onde ficam

| # | Item | Onde |
|---|---|---|
| 1 | Middleware que intercepta requisições | `@app.middleware("http")` no `main.py` |
| 2 | Extrai IP, rota, usuário (JWT), data/hora | dentro do middleware (`request.client.host`, `request.url.path`, `jwt.decode`, `func.now()`) |
| 3 | Salva numa tabela de Logs | `models/log_model.py` + o `db.add(Log(...))` |

## Cuidados / observações

- O middleware roda em **toda** requisição — pra não pesar, a gente filtra só os métodos que modificam.
- O `POST /login` também é logado (é uma ação), mas com `usuario_id` nulo (ainda não tem token na requisição). Faz sentido: registra "tentativa de login do IP X".
- Criar uma sessão do banco por requisição tem um custo. Pra um projeto acadêmico, tudo bem; em produção, dá pra otimizar depois.

## Links

- [[RF004 - Registro de implementação (o que fizemos)]] - a explicação middleware x dependency
- [[Proteger Rotas - Registro de implementação (o que fizemos)]] - o JWT que a gente decodifica aqui
- [[Modelagem do Banco de Dados]] - a tabela logs
- Card: [NF006] Middleware de Logs de Auditoria
