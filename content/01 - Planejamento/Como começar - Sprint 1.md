---
title: Como começar — Sprint 1
tags: [sprint-1, setup, grupo-5]
---

# Como começar — Sprint 1

Guia rápido de setup do ambiente e do que precisa rolar primeiro. Lê antes de pegar qualquer card.

> [!important] Antes de tudo, lê a [[00 - LEIA PRIMEIRO - Repositórios novos do professor]]
> - Nosso código vai no monorepo do grupo: **https://github.com/hiag-code/poswebreactdeploy** (pasta `back/` = FastAPI, `src/` = React)
> - Backend de **referência** do professor (Node + JSON, serve de mapa): https://github.com/DjanInfo/poswebserver
>
> O ambiente **já vem montado** no repo: `requirements.txt`, a conexão do banco (`back/db/database.py`) e as pastas (`models/`, `routes/`, `schemas/`). Não cria do zero — clona o repo e preenche o que falta.

---

## Ordem em que as coisas têm que rolar

```mermaid
flowchart TD
    A[Configurar Ambiente] --> B[Modelagem - Usuario primeiro]
    B --> C[RF001 Login JWT]
    B --> D[Modelagem completa]
    D --> E[RF002 Cadastro Aluno]
    D --> F[RF003 Cadastro Docente]
    C --> E
    C --> F

    style A fill:#2d4a2d
    style C fill:#4a3a2d
    style E fill:#2d3a4a
    style F fill:#2d3a4a
```

A pegadinha: o RF001 (login) precisa do model `Usuario` existindo. Por isso a modelagem faz primeiro o `Usuario` e libera o RF001 antes de terminar os outros models.

---

## Decisões já fechadas

- [x] Onde fica o código → pasta `back/` do repo `hiag-code/poswebreactdeploy`
- [x] Banco → PostgreSQL (local em dev)
- [x] JWT_SECRET → gerado e no `.env` (não vai pro Git)
- [x] CORS → `djansantos.com.br` (produção) + `localhost:5173` (Vite em dev)

---

## Passo a passo do Card 1 (Configurar Ambiente)

O repositório já vem montado, então "configurar ambiente" é mais **clonar e rodar** do que criar do zero.

### 1. Clonar o monorepo e criar a branch
```bash
git clone https://github.com/hiag-code/poswebreactdeploy.git
cd poswebreactdeploy
git checkout -b feature/configurar-ambiente
```

### 2. Backend (FastAPI) — venv + instalar deps
```bash
python -m venv .venv
.venv\Scripts\activate          # Windows
pip install -r requirements.txt   # já vem no repo
```

### 3. Frontend (React) — quando for mexer no front
```bash
npm install                       # lê o package.json
```

> [!note] Os "dois tipos de arquivos"
> O monorepo tem dois mundos: `pip install -r requirements.txt` (FastAPI) e `npm install` (React). Pra rodar o projeto inteiro, instala os dois. Pra mexer só no backend, basta o `pip`.

### 4. Estrutura que já vem no repo (`back/`)
```
poswebreactdeploy/
├── src/                     ← React (front)
├── back/                    ← FastAPI (nosso trabalho)
│   ├── main.py              (mínimo, falta CORS)
│   ├── db/database.py       engine + SessionLocal + Base + get_db prontos
│   ├── models/user_model.py ← VAZIO — a modelagem começa aqui
│   ├── routes/              (vazio)
│   └── schemas/             (vazio)
├── requirements.txt         já existe
└── .env
```

> [!warning] Não recrie o que já existe
> Não crie outro `requirements.txt`, nem outro `database.py`, nem pasta `app/`. Use o que já vem montado. Seguir a estrutura antiga (`app/`) duplica tudo.

### 5. Configurar o `.env` (na raiz, não commitar)
```env
DATABASE_URL=postgresql://user:senha@localhost:5432/posweb
JWT_SECRET=troca-isso
JWT_ALGORITHM=HS256
JWT_EXPIRE_MINUTES=60
```
Gera o secret: `python -c "import secrets; print(secrets.token_urlsafe(32))"`

### 6. Adicionar CORS no `back/main.py`
Falta o CORS pro front conseguir conversar:
```python
from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware

app = FastAPI(title="API Pós-graduação IFBA")

app.add_middleware(
    CORSMiddleware,
    allow_origins=[
        "https://djansantos.com.br",   # produção
        "http://localhost:5173",       # Vite em dev (React)
    ],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

@app.get("/")
def root():
    return {"status": "ok"}
```

### 7. Rodar e testar
```bash
cd back
uvicorn main:app --reload
```
Abre http://localhost:8000/docs — se o Swagger aparecer, **funcionou**.

> [!tip] Alembic (migrations)
> Adotamos o Alembic pra versionar o banco (`alembic init`, `revision --autogenerate`, `upgrade head`). Detalhe na [[Modelagem do Banco de Dados]].

---

## O que vem depois do ambiente subir

### Modelagem
1. Preenche `back/models/user_model.py` com o model `Usuario` — import `from db.database import Base`
2. Configura Alembic e gera a primeira migration
3. Faz os outros models em `back/models/` usando os **JSON do professor como base**: Aluno (`alunos.json`), Docente (`docentes.json`), Disciplina (`disciplinas.json`)

Passo a passo completo na nota [[Modelagem do Banco de Dados]].

### RF001 Login
Depois do Usuario pronto (dentro de `back/`):
1. `back/core/security.py` → `hash_password()`, `verify_password()` (passlib) e `create_access_token()` (python-jose)
2. `back/schemas/auth.py` → `LoginRequest`, `TokenResponse`
3. `back/routes/auth.py` → `POST /auth/login`
4. Inclui no `back/main.py`: `app.include_router(auth_router)`

Passo a passo completo na nota [[RF001 - Login com JWT]].

### Cadastros (RF002, RF003)
Dependem do RF001 (precisam do `hash_password()`). Mesmo padrão: schema Pydantic → rota POST em `back/routes/` → tratar duplicado (409).

---

## Boas práticas

> [!warning] O que NÃO fazer
> - Comitar `.env` (vaza credencial)
> - Comitar `.venv/` (peso desnecessário)
> - Mexer direto na `main` — sempre branch + PR
> - Salvar senha em texto puro no banco
> - Mudar schema sem gerar migration no Alembic

> [!success] O que fazer
> - Mover o card no Trello: `Sprint 1` → `em andamento` → `em revisão` → `concluído`
> - Sempre branch + PR

---

## `.gitignore` recomendado (FastAPI)

```gitignore
.venv/
__pycache__/
*.pyc
.env
.env.local
*.db
.pytest_cache/
.coverage
```

---

## Links úteis

- [[00 - LEIA PRIMEIRO - Repositórios novos do professor]] — contexto da troca de repos
- Backend referência (Node): https://github.com/DjanInfo/poswebserver
- Frontend (React/Vite): https://github.com/DjanInfo/poswebreactdeploy
- Site no ar: https://pos-web-ifba.vercel.app/
- FastAPI docs: https://fastapi.tiangolo.com/
- SQLAlchemy 2: https://docs.sqlalchemy.org/en/20/
- JWT no FastAPI: https://fastapi.tiangolo.com/tutorial/security/oauth2-jwt/
- Trello: https://trello.com/b/ewib1PdF
