---
title: RF003 - Cadastro de Docente (guia)
tags: [rf003, docente, professor, cadastro, guia, sprint-1, backend]
status: a-fazer
sprint: 1
---

# RF003 - Cadastro de Professor/Docente (guia)

> Status: a fazer.
> Boa notícia: é quase idêntico ao RF002. Você já domina o padrão - só muda o que cada um guarda.

Guia pra implementar o cadastro de docente. Lê antes de codar.

---

## O que o card pede (checklist do Trello)

1. Criar schema do Pydantic pra validar os dados (Nome, CPF, Titulação, Email, Senha)
2. Implementar rota POST /professores com hash de senha
3. Tratar exceção (HTTP 422) para dados inválidos ou duplicados

---

## Duas coisas pra decidir antes (análise crítica - o card não é absoluto)

> Lembrando: os cards foram feitos por um colega, não pelo professor. Onde o card pede algo estranho, a gente analisa.

### 1. "Professor" vs "Docente"
O card diz "professor" / rota "/professores". Mas o nosso **model** e o **backend de referência do professor** chamam de **docente**. 
- **Recomendação:** usar o model `Docente` e a rota **`/docentes`** (consistente com o resto). Se o front exigir `/professores`, dá pra usar esse nome na rota - alinhar com o grupo.

### 2. Código de erro: 422 vs 409 para duplicado
O card diz "HTTP 422 para dados inválidos ou duplicados". Mas:
- **422** é o código de **validação** (dado mal formatado) - e o Pydantic já devolve isso sozinho.
- **409 (Conflict)** é a convenção pra **duplicado** (CPF/email já existe) - foi o que usamos no RF002.
- **Recomendação:** usar **409 para duplicado** (igual RF002, mantém consistência) e deixar o **422 vir de graça** do Pydantic pros inválidos. Alinhar com o grupo, mas misturar duplicado em 422 não é padrão.

---

## É o mesmo padrão do RF002 (Opção A)

Igual ao aluno: cadastrar um docente **cria o login dele junto**.
```
recebe { nome, cpf, titulacao, email, senha }
        ↓
cria USUARIO (login, role "docente", senha em hash)  +  DOCENTE (dados), ligados
```
A única diferença pro aluno: aqui tem **titulacao** no lugar de **data_nascimento**, e a role é **docente**.

---

## O que falta no model Docente

Hoje o `Docente` tem: id, nome, email, status, usuario_id. Faltam:
- `cpf`
- `titulacao`

---

## O plano (4 etapas, igual RF002)

```
1. Model    -> adiciona cpf e titulacao ao Docente
2. Alembic  -> migration pras colunas novas
3. Schema   -> DocenteCreate e DocenteResponse
4. Rota     -> POST /docentes (Opção A) + ligar no main.py
```

---

## Passo a passo (código)

### Etapa 1 - Model `back/models/docente_model.py`
Adiciona:
```python
cpf = Column(String(14), unique=True, nullable=False)
titulacao = Column(String(100), nullable=True)
```
(o `String` já deve estar importado; confere)

### Etapa 2 - Migration
De dentro de `back/`:
```powershell
alembic revision --autogenerate -m "adiciona cpf e titulacao ao docente"
alembic upgrade head
```
Abre o arquivo gerado e confere que tem `op.add_column('docentes', ...)` pras duas colunas.

### Etapa 3 - Schema `back/schemas/docente_schema.py` (arquivo novo)
```python
from pydantic import BaseModel, ConfigDict, Field


class DocenteCreate(BaseModel):
    nome: str = Field(min_length=1)
    cpf: str = Field(min_length=11, max_length=14)
    titulacao: str = Field(min_length=1)
    email: str = Field(min_length=1)
    senha: str = Field(min_length=4)


class DocenteResponse(BaseModel):
    model_config = ConfigDict(from_attributes=True)

    id: int
    nome: str
    cpf: str
    titulacao: str | None
    email: str
    status: str
```

### Etapa 4 - Rota `back/routes/docente_routes.py` (arquivo novo)
```python
from fastapi import APIRouter, Depends, HTTPException, status
from sqlalchemy.orm import Session

from db.database import get_db
from models.docente_model import Docente
from models.user_model import Usuario
from schemas.docente_schema import DocenteCreate, DocenteResponse
from core.security import hash_password

router = APIRouter(prefix="/docentes", tags=["Docentes"])


@router.post("/", response_model=DocenteResponse, status_code=status.HTTP_201_CREATED)
def criar_docente(dados: DocenteCreate, db: Session = Depends(get_db)):
    # 1. checa duplicado (CPF ou email)
    if db.query(Docente).filter((Docente.cpf == dados.cpf) | (Docente.email == dados.email)).first():
        raise HTTPException(status_code=409, detail="CPF ou email ja cadastrado")
    if db.query(Usuario).filter(Usuario.email == dados.email).first():
        raise HTTPException(status_code=409, detail="Email ja cadastrado")

    # 2. cria o login (Usuario) com a senha em hash, role docente
    usuario = Usuario(
        email=dados.email,
        senha_hash=hash_password(dados.senha),
        role="docente",
    )
    db.add(usuario)
    db.flush()

    # 3. cria o Docente ligado ao login
    docente = Docente(
        nome=dados.nome,
        cpf=dados.cpf,
        titulacao=dados.titulacao,
        email=dados.email,
        usuario_id=usuario.id,
    )
    db.add(docente)
    db.commit()
    db.refresh(docente)
    return docente


@router.get("/", response_model=list[DocenteResponse])
def listar_docentes(db: Session = Depends(get_db)):
    return db.query(Docente).all()
```

### Etapa 5 - Ligar no `back/main.py`
No import das rotas, adiciona `docente_routes`:
```python
from routes import aluno_routes, auth_routes, docente_routes
...
app.include_router(docente_routes.router)
```

---

## Os itens do checklist -> onde ficam

| # | Item | Onde |
|---|---|---|
| 1 | Schema (Nome, CPF, Titulação, Email, Senha) | DocenteCreate |
| 2 | Rota POST com hash | docente_routes.py + hash_password |
| 3 | Tratar inválido/duplicado | 422 (Pydantic, automático) + 409 (duplicado) |

---

## Como testar (depois)

1. `alembic upgrade head`
2. `uvicorn main:app --reload`  ->  `/docs`
3. POST /docentes com nome, cpf, titulacao, email, senha -> 201
4. POST /login com o email/senha do docente -> 200 (login nasce junto, role docente)
5. Repetir -> 409
6. Senha vazia -> 422

---

## Decisões pra confirmar com o grupo

- [ ] Nome da rota: /docentes ou /professores?
- [ ] Código pra duplicado: 409 (recomendado) ou 422 (como o card diz)?
- [ ] Titulação obrigatória no cadastro? (o guia deixa obrigatória)

---

## Links

- [[RF002 - Registro de implementação (o que fizemos)]] - o padrão que estamos repetindo
- [[RF002 - Cadastro de Aluno (guia)]] - o guia gêmeo (aluno)
- [[Estrutura de Pastas do Backend]] - onde cada arquivo fica
- Card no Trello: [RF003] Cadastro de Professor (POST /professores)
