---
title: RF002 - Cadastro de Aluno (guia)
tags: [rf002, aluno, cadastro, guia, sprint-1, backend]
status: a-fazer
sprint: 1
---

# RF002 — Cadastro de Aluno (guia pra quando formos fazer)

> **Status:** a fazer.
> **Decisão tomada:** **Opção A** — cadastrar um aluno **cria o login (Usuario) junto**.

Guia completo pra implementar o RF002 do jeito certo. Lê antes de codar.

---

## O que o card pede (checklist do Trello)

1. Criar schema do Pydantic pra validar a entrada: **Nome, CPF, Email, Data de Nasc, Senha**
2. Implementar a função de **hash de senha** (passlib)
3. Criar rota **POST /alunos** pra inserir no banco *(já temos uma versão básica)*
4. Tratar exceção (**409/400**) se o **CPF ou Email** já existir no banco

> Repara: o item 3 já está marcado porque a gente fez um `/alunos` simples. Mas ele precisa ser **atualizado** pra incluir os campos novos.

---

## A decisão: Opção A (cadastro cria o login junto)

O card pede **Senha** no cadastro de aluno. Como na nossa arquitetura senha mora na tabela **`usuarios`** (login), o `POST /alunos` vai:

```
recebe { nome, cpf, email, data_nascimento, senha }
        ↓
1. cria um USUARIO (login):  email + senha em hash + role "estudante"
2. cria um ALUNO:            nome, cpf, email, data_nascimento + usuario_id (liga ao login)
        ↓
devolve o aluno criado (SEM a senha)
```

**Por que assim:** respeita a modelagem (login separado dos dados acadêmicos), reaproveita o `hash_password` que já foi feito no RF001, e usa a relação `aluno.usuario_id → usuarios.id`.

---

## O que já temos vs o que falta

| | Temos hoje | Falta pro RF002 |
|---|---|---|
| Campos | nome, email, matricula, status | **cpf, data_nascimento, senha** |
| Hash da senha | | usar `hash_password` |
| Criar o login junto | | criar Usuario |
| Checar duplicado | só email | **cpf + email** |

---

## O plano (4 etapas)

```
1. Model    → adiciona cpf e data_nascimento ao Aluno
2. Alembic  → gera migration pros campos novos  (USO REAL DO ALEMBIC!)
3. Schema   → AlunoCreate com os campos do card
4. Rota     → hash + cria usuario + cria aluno + checa duplicado
```

---

## Passo a passo (pra quando for fazer)

### Etapa 1 — Atualizar o model `back/models/aluno_model.py`

Adiciona duas colunas:
```python
from sqlalchemy import Column, Integer, String, Date, ForeignKey
from sqlalchemy.orm import relationship
from db.database import Base


class Aluno(Base):
    __tablename__ = "alunos"

    id = Column(Integer, primary_key=True, index=True)
    matricula = Column(String(20), unique=True, nullable=True)   # ← virou nullable (ver decisão abaixo)
    nome = Column(String(255), nullable=False)
    cpf = Column(String(14), unique=True, nullable=False)        # ← NOVO
    email = Column(String(255), unique=True, nullable=False)
    data_nascimento = Column(Date, nullable=True)                # ← NOVO
    status = Column(String(20), default="Ativo")
    data_cadastro = Column(Date)
    usuario_id = Column(Integer, ForeignKey("usuarios.id"), nullable=True)

    matriculas = relationship("Matricula", back_populates="aluno")
```

> **Decisão sobre `matricula`:** o card não pede matrícula na entrada. Por isso deixei ela `nullable=True` (opcional). Alternativa: gerar automática (ex: ano + sequência). Decide com o grupo.

### Etapa 2 — Gerar a migration com Alembic (o uso real!)

Esse é o momento que o Alembic brilha — ele vai fazer `ALTER TABLE` pra adicionar as colunas **sem apagar** os dados. De dentro de `back/`:
```powershell
alembic revision --autogenerate -m "adiciona cpf e data_nascimento ao aluno"
alembic upgrade head
```
Abre o arquivo gerado em `alembic/versions/` → vai ter `op.add_column('alunos', ...)`. **Isso é o que o create_tables.py nunca conseguiu fazer.**

### Etapa 3 — Atualizar o schema `back/schemas/aluno_schema.py`

```python
from datetime import date
from pydantic import BaseModel, ConfigDict


class AlunoCreate(BaseModel):
    nome: str
    cpf: str
    email: str
    data_nascimento: date
    senha: str


class AlunoResponse(BaseModel):
    model_config = ConfigDict(from_attributes=True)

    id: int
    nome: str
    cpf: str
    email: str
    data_nascimento: date
    # NUNCA inclui senha aqui — resposta não devolve senha nem hash
```

### Etapa 4 — Atualizar a rota `back/routes/aluno_routes.py`

```python
from fastapi import APIRouter, Depends, HTTPException, status
from sqlalchemy.orm import Session

from db.database import get_db
from models.aluno_model import Aluno
from models.user_model import Usuario
from schemas.aluno_schema import AlunoCreate, AlunoResponse
from core.security import hash_password

router = APIRouter(prefix="/alunos", tags=["Alunos"])


@router.post("/", response_model=AlunoResponse, status_code=status.HTTP_201_CREATED)
def criar_aluno(dados: AlunoCreate, db: Session = Depends(get_db)):
    # [item 4] checa CPF ou email já cadastrado
    if db.query(Aluno).filter((Aluno.cpf == dados.cpf) | (Aluno.email == dados.email)).first():
        raise HTTPException(status_code=409, detail="CPF ou email já cadastrado")
    if db.query(Usuario).filter(Usuario.email == dados.email).first():
        raise HTTPException(status_code=409, detail="Email já cadastrado")

    # [item 2] cria o login (Usuario) com a senha em HASH
    usuario = Usuario(
        email=dados.email,
        senha_hash=hash_password(dados.senha),
        role="estudante",
    )
    db.add(usuario)
    db.flush()   # envia o insert e preenche usuario.id, SEM commitar ainda

    # cria o Aluno ligado ao login
    aluno = Aluno(
        nome=dados.nome,
        cpf=dados.cpf,
        email=dados.email,
        data_nascimento=dados.data_nascimento,
        usuario_id=usuario.id,
    )
    db.add(aluno)
    db.commit()      # grava os DOIS de uma vez (transação)
    db.refresh(aluno)
    return aluno
```

(o `GET /alunos` continua igual, só listando)

---

## Conceitos novos que aparecem aqui

| Conceito | O que é | Por que |
|---|---|---|
| `db.flush()` | "empurra" o insert pro banco **sem** commitar, e preenche o `id` | a gente precisa do `usuario.id` pra ligar o aluno, **antes** de fechar a transação |
| **transação** | os 2 inserts (usuario + aluno) viram **um só** commit | se o aluno falhar, o usuario também não é gravado — não fica login órfão |
| nunca devolver senha | o `AlunoResponse` não tem campo senha | resposta de API nunca expõe senha nem hash |

---

## Os 4 itens do checklist → onde ficam

| # | Item | Onde |
|---|---|---|
| 1 | Schema com Nome, CPF, Email, Data Nasc, Senha | `AlunoCreate` (etapa 3) |
| 2 | Hash da senha | `hash_password(dados.senha)` na rota (etapa 4) |
| 3 | Rota POST /alunos | `aluno_routes.py` (etapa 4) |
| 4 | 409 se CPF ou email existir | os dois `if ... raise HTTPException(409)` (etapa 4) |

---

## Como testar (depois)

1. `alembic upgrade head` (aplica os campos novos)
2. `uvicorn main:app --reload` → `/docs`
3. `POST /alunos`:
   ```json
   {
     "nome": "Maria Silva",
     "cpf": "12345678900",
     "email": "maria@ifba.edu.br",
     "data_nascimento": "2000-05-15",
     "senha": "maria123"
   }
   ```
   → **201** com o aluno (sem senha)
4. Testa **duplicado**: manda o mesmo CPF/email de novo → **409**
5. Confere no banco: a tabela `alunos` tem a Maria E a tabela `usuarios` tem o login dela (role estudante)
6. Bônus: tenta **logar** com `maria@ifba.edu.br / maria123` no `/login` → deve dar 200 (o cadastro criou o login!)

---

## Decisões pra confirmar com o grupo

- [ ] **Matrícula:** auto-gerar, deixar opcional, ou pedir na entrada? (o card não pede)
- [ ] **Email no aluno E no usuario:** guardar nos dois (contato + login) ou só no usuario? (o guia guarda nos dois, mais simples)
- [ ] **Validar formato do CPF/email?** (dá pra usar Pydantic `EmailStr` se instalar `email-validator`, e um validator de CPF)

---

## Links

- [[RF001 - Registro de implementação (o que fizemos)]] — o padrão que seguimos
- [[Modelagem do Banco de Dados]] — os models
- [[Conceitos Python do Código (def, return, etc)]] — os termos do código
- Card no Trello: [RF002] Cadastro de Aluno (POST /alunos)
