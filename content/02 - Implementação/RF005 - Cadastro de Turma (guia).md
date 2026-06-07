---
title: RF005 - Cadastro de Turma (guia)
tags: [rf005, turma, fastapi, jwt, autorizacao, sprint-2, guia]
status: em andamento
sprint: 2
card_trello: "[RF005] Cadastro de Turma (POST /turmas)"
---

# RF005 - Cadastro de Turma (guia)

> Status: em andamento (uma primeira versão foi feita; em revisão).
> Rota pra registrar uma turma = uma disciplina + um docente + semestre + vagas. Protegida (admin).

## O que o card pede

1. Criar schema Pydantic (ID da Disciplina, ID do Professor, Semestre, Vagas Iniciais)
2. Proteger a rota com JWT
3. Verificar se os IDs do Professor e da Disciplina existem no banco (404 se não)
4. Salvar a turma e retornar HTTP 201

## O que é uma Turma (no nosso modelo)

A tabela `turmas` **já existe** (da modelagem inicial): `id`, `disciplina_id` (FK), `docente_id` (FK), `semestre`, `vagas_total`. Uma turma é uma **oferta** de uma disciplina, com um professor, num semestre.

> [!info] Uma disciplina tem VÁRIAS turmas
> É um relacionamento um-pra-muitos: a mesma disciplina pode ter turmas em semestres diferentes ou com professores diferentes. **Não dá pra travar "1 turma por disciplina".**

## Conceito novo: validar chave estrangeira (FK) antes de salvar

A turma aponta pra uma disciplina e um docente **por id**. Antes de salvar, a gente confere se esses ids **existem de verdade** no banco — senão devolve **404**.

Por que checar? Se não checar, o banco até barraria (erro de chave estrangeira), mas com uma mensagem feia de erro 500. Checando no código, a gente devolve um **404 claro** ("disciplina não encontrada").

## Decisões

> [!check] Proteção: `require_admin` (não `get_current_user`)
> O card só diz "proteger com JWT". Mas criar turma é tarefa da **coordenação** — o time combinou deixar **só admin**, igual no RF004. Então usa `require_admin` (a secretaria usa contas admin).

> [!note] Sem migration
> A tabela `turmas` já existe — o RF005 **não precisa** de migration nova.

## Passo a passo

### 1. Schema (`schemas/turma_schema.py`)
```python
from pydantic import BaseModel, ConfigDict, Field


class TurmaCreate(BaseModel):
    disciplina_id: int = Field(ge=1)
    docente_id: int = Field(ge=1)
    semestre: str = Field(min_length=1, max_length=10)
    vagas_total: int = Field(ge=1)


class TurmaResponse(BaseModel):
    model_config = ConfigDict(from_attributes=True)

    id: int
    disciplina_id: int
    docente_id: int
    semestre: str
    vagas_total: int
```
- `ge=1` = "greater or equal 1" (id e vagas têm que ser >= 1).
- `min_length/max_length` no semestre (ex: "2026.1").

### 2. Rota (`routes/turma_routes.py`)
```python
from fastapi import APIRouter, Depends, HTTPException, status
from sqlalchemy.orm import Session

from db.database import get_db
from models.turma_model import Turma
from models.disciplina_model import Disciplina
from models.docente_model import Docente
from schemas.turma_schema import TurmaCreate, TurmaResponse
from core.security import require_admin

router = APIRouter(prefix="/turmas", tags=["Turmas"])


@router.post("/", response_model=TurmaResponse, status_code=status.HTTP_201_CREATED)
def criar_turma(
    dados: TurmaCreate,
    db: Session = Depends(get_db),
    usuario: dict = Depends(require_admin),
):
    # item 3: a disciplina existe?
    if not db.query(Disciplina).filter(Disciplina.id == dados.disciplina_id).first():
        raise HTTPException(status_code=status.HTTP_404_NOT_FOUND, detail="Disciplina nao encontrada")

    # item 3: o docente existe?
    if not db.query(Docente).filter(Docente.id == dados.docente_id).first():
        raise HTTPException(status_code=status.HTTP_404_NOT_FOUND, detail="Docente nao encontrado")

    # item 4: salva e devolve 201
    turma = Turma(**dados.model_dump())
    db.add(turma)
    db.commit()
    db.refresh(turma)
    return turma
```

### 3. Registrar no `main.py`
```python
from routes import aluno_routes, auth_routes, docente_routes, me_routes, disciplina_routes, turma_routes
# ...
app.include_router(turma_routes.router)
```

## Pontos de revisão (sobre a versão que já foi feita)

> [!warning] O que ajustar antes de mergear
> 1. Trocar `get_current_user` por `require_admin` (decisão do time).
> 2. **Remover** o `409 "disciplina já possui turma"** — uma disciplina tem várias turmas; esse check trava o RF006.
> 3. Atualizar a branch com a `main` antes (pra ter o `require_admin` e evitar conflito no `main.py`).

## Os 4 itens do checklist -> onde ficam

| # | Item | Onde |
|---|---|---|
| 1 | Schema (ids, semestre, vagas) | `turma_schema.py` |
| 2 | Proteção JWT (só admin) | `require_admin` na rota |
| 3 | IDs existem -> 404 | checagens de Disciplina/Docente |
| 4 | Salvar -> 201 | `add/commit/refresh` + `status_code=201` |

## Links

- [[RF004 - Registro de implementação (o que fizemos)]] - o `require_admin` que reusamos
- [[RF006 - Efetuar Matrícula (guia)]] - o próximo, que usa as turmas
- [[Modelagem do Banco de Dados]] - as tabelas turmas/matriculas
- Card: [RF005] Cadastro de Turma (POST /turmas)
