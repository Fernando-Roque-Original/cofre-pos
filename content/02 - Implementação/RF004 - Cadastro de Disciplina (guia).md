---
title: RF004 - Cadastro de Disciplina (guia)
tags: [rf004, disciplina, fastapi, autorizacao, jwt, sprint-2, guia]
status: em andamento
sprint: 2
card_trello: "[RF004] Cadastro de Disciplina (POST /disciplinas)"
---

# RF004 - Cadastro de Disciplina (guia)

> Status: em andamento (Sprint 2).
> Rota pra coordenação registrar as matérias do curso. Protegida por JWT e restrita a admin.

## O que o card pede

Descrição: "Rota para a coordenação registrar as matérias do curso. Precisa estar protegida pelo token JWT."

Checklist:
1. Criar schema Pydantic pra validar entrada (Nome, Código, Carga Horária, Ementa).
2. Aplicar o middleware de JWT pra proteger a rota (apenas secretaria/admin).
3. Validar no banco se o "Código" da disciplina já existe (HTTP 409 se existir).
4. Salvar a disciplina e retornar HTTP 201 (Created).

## Antes de tudo: 2 coisas do card que NÃO batem com o nosso código

(o card foi feito por um colega; a gente sempre confere com a realidade do projeto)

> [!check] 1. O model de Disciplina não tinha "Código" (decisão: adicionar)
> O nosso `models/disciplina_model.py` tem: `nome`, `ementa`, `carga_horaria`, `link`. **Não existe `codigo`**.
> Mas o card pede o campo "Código" e ainda quer checar duplicado por ele (item 3).
> **Decisão (confirmada):** ADICIONAR a coluna `codigo` (única) na tabela — disciplina de verdade tem código mesmo (tipo "BD101"). É uma mudança de model + migration (Passo 0). Confirmado também pelo RF005, que aponta pra disciplina por id.

> [!check] 2. "Secretaria" = conta admin (confirmado)
> Os papéis do sistema (no `user_model.py`) são: **estudante | docente | admin**. Não tem "secretaria".
> O card diz "apenas secretaria/admin". **Confirmado:** a secretaria usa contas com role **admin** — então a regra **"apenas admin"** já cobre secretaria + coordenação. Resolvido.

## O que vamos construir (visão geral)

```
Passo 0  models/disciplina_model.py   -> adicionar coluna `codigo` (+ migration)
Passo 1  schemas/disciplina_schema.py -> DisciplinaCreate + DisciplinaResponse
Passo 2  core/security.py             -> require_admin (autorização por papel)  [NOVO]
Passo 3  routes/disciplina_routes.py  -> POST /disciplinas (protegida + 409 + 201)
Passo 4  main.py                      -> registrar o router
Passo 5  testar
```

## Conceito novo: autenticação x autorização

A gente já tem **autenticação** (o porteiro `get_current_user`: "quem é você?" — o token é válido?).
Agora entra a **autorização**: "você PODE fazer isso?" — você é admin?

| | Pergunta | Quem faz | Erro se falhar |
|---|---|---|---|
| Autenticação | Você está logado? (token válido) | `get_current_user` | 401 |
| Autorização | Você tem permissão? (role == admin) | `require_admin` | 403 |

- **401** = não autenticado (sem token ou token ruim)
- **403** = autenticado, mas sem permissão (logado, mas não é admin)

## Passo 0 — Adicionar a coluna `codigo` (model + migration)

No `models/disciplina_model.py`, adicione a coluna:
```python
codigo = Column(String(50), unique=True, nullable=False, index=True)
```
O `unique=True` é o que não deixa repetir — é ele que vai permitir o 409 do item 3.

Depois, na pasta `back/`, gere e aplique a migration (igual fizemos no RF002/RF003):
```bash
alembic revision --autogenerate -m "add codigo na disciplina"
alembic upgrade head
```

> [!note] Atenção se a tabela já tiver dados
> Adicionar uma coluna `NOT NULL` numa tabela que já tem linhas dá erro (as linhas antigas ficariam sem código). Se a tabela `disciplinas` estiver vazia (provável, projeto novo), tudo certo. Se tiver dados, a gente põe um default temporário.

## Passo 1 — Schema (schemas/disciplina_schema.py)

Mesma ideia do `aluno_schema.py`: um pra ENTRADA, um pra SAÍDA.
```python
from pydantic import BaseModel, ConfigDict, Field

# o que o cliente ENVIA pra criar
class DisciplinaCreate(BaseModel):
    nome: str = Field(min_length=2)
    codigo: str = Field(min_length=2)
    carga_horaria: int = Field(gt=0)     # tem que ser maior que 0
    ementa: str | None = None            # opcional

# o que a API DEVOLVE (com o id gerado)
class DisciplinaResponse(BaseModel):
    model_config = ConfigDict(from_attributes=True)
    id: int
    nome: str
    codigo: str
    carga_horaria: int
    ementa: str | None = None
```
- `Field(min_length=2)` e `Field(gt=0)` = a validação que o item 1 pede.
- `from_attributes=True` deixa devolver direto o objeto do SQLAlchemy.

## Passo 2 — A dependency de autorização (core/security.py)  [NOVO]

No `security.py` (onde já está o `get_current_user`), adicione:
```python
def require_admin(usuario: dict = Depends(get_current_user)):
    # require_admin DEPENDE de get_current_user:
    # primeiro valida o token, depois checa o papel
    if usuario["role"] != "admin":
        raise HTTPException(
            status_code=status.HTTP_403_FORBIDDEN,
            detail="Acesso restrito (apenas admin)",
        )
    return usuario
```
Repare: `require_admin` usa `Depends(get_current_user)` dentro dele. A ordem fica:
1. `get_current_user` valida o token (senão -> 401)
2. `require_admin` checa se é admin (senão -> 403)

É o nosso porteiro do "Proteger Rotas", agora com um segurança a mais conferindo o crachá.

## Passo 3 — A rota (routes/disciplina_routes.py)

Mesma estrutura do `aluno_routes.py`, com a proteção + a checagem de código:
```python
from fastapi import APIRouter, Depends, HTTPException, status
from sqlalchemy.orm import Session

from db.database import get_db
from models.disciplina_model import Disciplina
from schemas.disciplina_schema import DisciplinaCreate, DisciplinaResponse
from core.security import require_admin

router = APIRouter(prefix="/disciplinas", tags=["Disciplinas"])

@router.post("/", response_model=DisciplinaResponse, status_code=status.HTTP_201_CREATED)
def criar_disciplina(
    dados: DisciplinaCreate,
    db: Session = Depends(get_db),
    usuario: dict = Depends(require_admin),   # <- protege + exige admin
):
    # item 3: codigo duplicado -> 409
    existe = db.query(Disciplina).filter(Disciplina.codigo == dados.codigo).first()
    if existe:
        raise HTTPException(
            status_code=status.HTTP_409_CONFLICT,
            detail="Codigo de disciplina ja existe",
        )
    # item 4: salva e devolve 201
    disciplina = Disciplina(**dados.model_dump())
    db.add(disciplina)
    db.commit()
    db.refresh(disciplina)
    return disciplina
```

> [!note] Por que `Disciplina(**dados.model_dump())` funciona
> O `model_dump()` vira um dicionário `{nome, codigo, carga_horaria, ementa}`, e cada chave é uma coluna do model. O `link` (que o card não pede) fica nulo, e tudo bem (ele é `nullable=True`).

## Passo 4 — Registrar no main.py

```python
from routes import disciplina_routes
# ...
app.include_router(disciplina_routes.router)
```

## Passo 5 — Testar (no /docs)

Precisa de um usuário **admin** (ex: admin@ifba.edu.br / admin123).

| Teste | Esperado |
|---|---|
| POST /disciplinas SEM token | 401 |
| POST com token de estudante/docente | 403 |
| POST com token de admin, dados ok | 201 + a disciplina |
| POST com o mesmo `codigo` de novo | 409 |
| POST com carga_horaria = 0 ou nome vazio | 422 (validação do Pydantic) |

Fluxo: POST /login (admin) -> copia o token -> botão "Authorize" -> cola -> testa o POST /disciplinas.

## Os 4 itens do checklist -> onde ficam

| # | Item | Onde |
|---|---|---|
| 1 | Schema Pydantic (Nome, Código, Carga, Ementa) | `disciplina_schema.py` (Passo 1) |
| 2 | Proteger com JWT (só admin) | `require_admin` + `Depends` na rota (Passos 2-3) |
| 3 | Código duplicado -> 409 | `db.query(...).filter(codigo)...` (Passo 3) |
| 4 | Salvar -> 201 | `db.add/commit` + `status_code=201` (Passo 3) |

## Links

- [[Proteger Rotas - Registro de implementação (o que fizemos)]] - o `get_current_user` que reusamos aqui
- [[RF002 - Cadastro de Aluno (guia)]] - o padrão de schema + rota + 409
- [[Modelagem do Banco de Dados]] - a tabela disciplinas
- [[Conceitos Python do Código (def, return, etc)]] - Depends, model_dump, etc.
- Card: [RF004] Cadastro de Disciplina (POST /disciplinas)
