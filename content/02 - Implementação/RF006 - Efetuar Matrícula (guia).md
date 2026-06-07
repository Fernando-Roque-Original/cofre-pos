---
title: RF006 - Efetuar Matrícula (guia)
tags: [rf006, matricula, fastapi, jwt, transacao, sprint-2, guia]
status: a fazer
sprint: 2
card_trello: "[RF006] Efetuar Matrícula (POST /matriculas)"
---

# RF006 - Efetuar Matrícula (guia)

> Status: a fazer (depende do RF005 estar na main).
> O **aluno logado** se matricula numa turma; o sistema controla as vagas.

## O que o card pede

1. Rota protegida `POST /matriculas` que recebe o ID da Turma
2. Pegar o ID do Aluno **de dentro do token JWT** (não confiar no id do corpo)
3. Regra 1: verificar se o aluno já está matriculado nessa turma (evitar duplicidade)
4. Regra 2: verificar se `vagas_disponiveis > 0`
5. Salvar a matrícula + fazer o UPDATE na Turma diminuindo 1 vaga
6. Retornar HTTP 400 se não houver vagas

## Diferença importante: aqui NÃO é admin

RF004 e RF005 são da coordenação (admin). O RF006 é o **próprio aluno** se matriculando. Então a proteção é `get_current_user` (qualquer usuário logado), e a gente descobre **quem é** pelo token. Não usa `require_admin`.

## 3 conceitos novos

1. **Identidade vem do token, não do corpo.** Segurança: o aluno não pode mandar "matricule o aluno 5". A gente pega **quem está logado** do token. Se confiasse no corpo, qualquer um matricularia qualquer um.
2. **Regras de negócio.** Duplicado e vagas não são validação de "formato" (isso é o Pydantic); são de **lógica** — exigem consultar o banco.
3. **Transação.** Salvar a matrícula **e** diminuir a vaga têm que acontecer **juntos** — ou os dois, ou nenhum. Um `commit` só já é uma transação: se algo falhar antes do commit, nada é gravado.

## 2 decisões pra fechar ANTES de codar

> [!warning] 1. `vagas_disponiveis` não existe no model
> A `turmas` só tem `vagas_total`. O card quer checar `vagas_disponiveis` e diminuir. Duas opções:
> - **(A) Adicionar a coluna `vagas_disponiveis`** (migration). No RF005, ao criar a turma, `vagas_disponiveis = vagas_total`. No RF006, checa e diminui. → segue o card ao pé da letra.
> - **(B) Calcular na hora:** `disponiveis = vagas_total - (nº de matrículas da turma)`. Sem coluna, sem UPDATE, sem risco de o número "desencontrar". → mais robusto.
>
> Recomendo combinar com o time. Este guia mostra a **opção A** (que é o que o card descreve). Se escolherem a B, troca o `if turma.vagas_disponiveis <= 0` por uma contagem e **não** decrementa nada.

> [!warning] 2. O token tem o id do Usuário, não do Aluno
> O token guarda `sub` = id do **Usuario** (o login). Mas a matrícula precisa do id do **Aluno** (tabela `alunos`). Eles são ligados por **email** (o cadastro de aluno cria um Usuario com o mesmo email). Então o caminho é: token → Usuario → achar o Aluno pelo email → usar `aluno.id`.
>
> (Vale confirmar esse vínculo com o time. A alternativa mais "limpa" seria ter um `usuario_id` na tabela `alunos`, mas isso seria outra migration.)

## Passo a passo (assumindo a opção A das vagas)

### Passo 0 — `vagas_disponiveis` + migration (se escolher A)
- No `turma_model.py`: `vagas_disponiveis = Column(Integer, nullable=False)`
- `alembic revision --autogenerate -m "add vagas_disponiveis na turma"` + `alembic upgrade head`
- No RF005 (criar turma): setar `vagas_disponiveis = dados.vagas_total` ao criar

### 1. Schema (`schemas/matricula_schema.py`)
```python
from pydantic import BaseModel, ConfigDict, Field


class MatriculaCreate(BaseModel):
    turma_id: int = Field(ge=1)   # SÓ a turma! o aluno vem do token


class MatriculaResponse(BaseModel):
    model_config = ConfigDict(from_attributes=True)

    id: int
    aluno_id: int
    turma_id: int
    status: str
```
> Repara: o `MatriculaCreate` tem **só** `turma_id`. O `aluno_id` **não** vem do cliente — é o item 2 (segurança).

### 2. Rota (`routes/matricula_routes.py`)
```python
from fastapi import APIRouter, Depends, HTTPException, status
from sqlalchemy.orm import Session

from db.database import get_db
from models.matricula_model import Matricula
from models.turma_model import Turma
from models.aluno_model import Aluno
from models.user_model import Usuario
from schemas.matricula_schema import MatriculaCreate, MatriculaResponse
from core.security import get_current_user

router = APIRouter(prefix="/matriculas", tags=["Matriculas"])


@router.post("/", response_model=MatriculaResponse, status_code=status.HTTP_201_CREATED)
def matricular(
    dados: MatriculaCreate,
    db: Session = Depends(get_db),
    logado: dict = Depends(get_current_user),
):
    # item 2: descobrir o ALUNO a partir do token (nao do corpo)
    usuario = db.query(Usuario).filter(Usuario.id == int(logado["id"])).first()
    aluno = db.query(Aluno).filter(Aluno.email == usuario.email).first() if usuario else None
    if aluno is None:
        raise HTTPException(status_code=status.HTTP_403_FORBIDDEN, detail="Apenas alunos podem se matricular")

    # a turma existe?
    turma = db.query(Turma).filter(Turma.id == dados.turma_id).first()
    if turma is None:
        raise HTTPException(status_code=status.HTTP_404_NOT_FOUND, detail="Turma nao encontrada")

    # Regra 1 (item 3): ja matriculado nessa turma?
    ja_existe = db.query(Matricula).filter(
        Matricula.aluno_id == aluno.id,
        Matricula.turma_id == turma.id,
    ).first()
    if ja_existe:
        raise HTTPException(status_code=status.HTTP_409_CONFLICT, detail="Voce ja esta matriculado nessa turma")

    # Regra 2 (itens 4 e 6): tem vaga?
    if turma.vagas_disponiveis <= 0:
        raise HTTPException(status_code=status.HTTP_400_BAD_REQUEST, detail="Sem vagas nessa turma")

    # item 5: salva a matricula E diminui 1 vaga -- JUNTOS (uma transacao)
    matricula = Matricula(aluno_id=aluno.id, turma_id=turma.id)
    turma.vagas_disponiveis -= 1
    db.add(matricula)
    db.commit()
    db.refresh(matricula)
    return matricula
```

**O que cada bloco faz:**
- **Achar o aluno pelo token** → item 2. Pega o `id` do token, carrega o Usuario, acha o Aluno pelo email. Se não for aluno → 403.
- **Turma existe?** → 404 se não.
- **Regra 1 (duplicado)** → busca matrícula com o mesmo aluno + mesma turma; se achar → 409.
- **Regra 2 (vagas)** → se `vagas_disponiveis <= 0` → 400 (item 6).
- **Salvar + diminuir** → cria a matrícula, faz `turma.vagas_disponiveis -= 1`, e **um `commit` só** grava os dois juntos. É a transação (item 5).

### 3. Registrar no `main.py`
```python
from routes import ..., matricula_routes
app.include_router(matricula_routes.router)
```

## Por que um `commit` só (a transação)

A gente muda duas coisas: cria a matrícula e diminui a vaga. Com **um único `commit`**, ou os dois entram no banco, ou nenhum (se der erro antes, faz rollback). Assim nunca acontece de ter uma matrícula sem a vaga ter diminuído (ou o contrário).

## Os 6 itens do checklist -> onde ficam

| # | Item | Onde |
|---|---|---|
| 1 | Rota protegida recebendo turma_id | `MatriculaCreate` + `Depends(get_current_user)` |
| 2 | Aluno vem do token | bloco "achar o aluno pelo token" |
| 3 | Não duplicar matrícula | busca aluno+turma -> 409 |
| 4 | Checar vagas > 0 | `if turma.vagas_disponiveis <= 0` |
| 5 | Salvar + diminuir 1 vaga | `add` + `-= 1` + um `commit` |
| 6 | 400 se sem vagas | o `raise` da Regra 2 |

## Como testar (depois de pronto)

1. Login como **aluno** -> token -> Authorize
2. `POST /matriculas` com `turma_id` válido -> 201 (e a `vagas_disponiveis` da turma cai 1)
3. `POST` na **mesma** turma de novo -> 409 (duplicado)
4. Encher a turma e tentar de novo -> 400 (sem vagas)
5. Login como docente/admin e tentar -> 403 (não é aluno)

## Links

- [[RF005 - Cadastro de Turma (guia)]] - cria as turmas que aqui a gente matricula
- [[Proteger Rotas - Registro de implementação (o que fizemos)]] - o `get_current_user`
- [[RF004 - Registro de implementação (o que fizemos)]] - padrão de rota + validações
- [[Modelagem do Banco de Dados]] - tabelas turmas/matriculas/alunos
- Card: [RF006] Efetuar Matrícula (POST /matriculas)
