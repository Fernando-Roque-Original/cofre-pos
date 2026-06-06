---
title: Como começar — Sprint 1
tags: [sprint-1, setup, grupo-5]
---

# Como começar — Sprint 1

Pra gente não ficar esbarrando um no outro e travar o projeto na primeira semana, montei esse guia rápido. Lê com calma antes de pegar qualquer card.

> [!important] Antes de tudo, lê a [[00 - LEIA PRIMEIRO - Repositórios novos do professor]]
> - Nosso código vai no monorepo do Hiago: **https://github.com/hiag-code/poswebreactdeploy** (pasta `back/` = FastAPI, `src/` = React)
> - Backend de **referência** do professor (Node + JSON, serve de mapa): https://github.com/DjanInfo/poswebserver
>
> O Hiago **já montou** o ambiente: `requirements.txt`, a conexão do banco (`back/db/database.py`) e as pastas (`models/`, `routes/`, `schemas/`). A gente não cria isso do zero — clona o repo dele e preenche o que falta.

---

## Quem pega o quê

Somos **8** (segundo o relatório de maio). Dá pra paralelizar bem. Sugestão de divisão respeitando quem já tá escalado no relatório:

| Quem | Frente | Depende de |
|---|---|---|
| Fernando + Felipe | Configurar Ambiente + Modelagem (vocês já estão na "definição de tecnologia") | nada |
| Milena | [RF001] Auth JWT | model Usuario |
| Nickolas | [RF002] Cadastro Aluno | model Aluno + hash |
| Hiago | [RF003] Cadastro Docente | model Docente + hash (o prof chama de **docente**) |
| Ana Clara Fernandes | [RF004] Cadastro Disciplina (Sprint 2) | model Disciplina |
| Ana Clara Marinho | Apoio modelagem + testes (pytest) | models prontos |
| Tamily | Protótipo Figma + Sprint 4 (front React) | roda em paralelo |

> [!note] Isso é sugestão
> Ajustem como quiser — o importante é cada um saber o que é seu e mover o card pra **"em andamento"** no Trello. Como somos 8 e os cards de Sprint 1 são ~6, sobra gente pra adiantar Sprint 2 (disciplinas, turmas) e os testes.

A dupla Ambiente+Modelagem é a mais crítica: enquanto o banco não existe, o RF001 e os cadastros não andam.

---

## Ordem que as coisas têm que rolar

```mermaid
flowchart TD
    A[Configurar Ambiente<br/>28/mai] --> B[Modelagem - Usuario primeiro<br/>parcial]
    B --> C[RF001 Login JWT<br/>31/mai]
    B --> D[Modelagem completa<br/>7/jun]
    D --> E[RF002 Cadastro Aluno<br/>14/jun]
    D --> F[RF003 Cadastro Docente<br/>14/jun]
    C --> E
    C --> F

    style A fill:#2d4a2d
    style C fill:#4a3a2d
    style E fill:#2d3a4a
    style F fill:#2d3a4a
```

A pegadinha: o RF001 (login) precisa do model `Usuario` existindo. Por isso o pessoal da Modelagem faz primeiro o `Usuario` e libera o RF001 antes de terminar os outros models.

---

## Decisões que precisamos fechar HOJE

Antes de qualquer um começar, alinhem isso no grupo:

- [x] ~~Onde fica nosso código?~~ → resolvido: pasta `back/` do repo `hiag-code/poswebreactdeploy`
- [ ] **Banco:** PostgreSQL ou MySQL? *(eu voto Postgres — free no Neon/Railway, mais comum em produção)*
- [ ] **Banco em dev:** cada um sobe o seu local (Docker) ou usamos um compartilhado (Neon)?
- [ ] **JWT_SECRET:** alguém gera e cola no `.env` compartilhado (não vai pro Git)
- [ ] **Domínio CORS:** `djansantos.com.br` (produção) + `localhost:5173` (Vite em dev — o front agora é React/Vite, porta 5173, não 3000)

---

## Passo a passo do Card 1 (Configurar Ambiente)

O Hiago já deixou o repositório montado, então "configurar ambiente" agora é mais **clonar e rodar** do que criar do zero.

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
pip install -r requirements.txt   # o Hiago JÁ criou esse arquivo
```

### 3. Frontend (React) — quando for mexer no front
```bash
npm install                       # lê o package.json
```

> [!note] É isso que o Hiago quis dizer com "dois tipos de arquivos"
> O monorepo tem dois mundos: `pip install -r requirements.txt` (FastAPI) e `npm install` (React). Pra rodar o projeto inteiro, instala os dois. Pra mexer só no backend, basta o `pip`.

### 4. Estrutura que o Hiago já deixou (`back/`)
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
> Não crie outro `requirements.txt`, nem outro `database.py`, nem pasta `app/`. Use o que o Hiago montou. Quem seguir a estrutura antiga (`app/`) vai duplicar tudo.

### 5. Configurar o `.env` (na raiz, não commitar)
```env
DATABASE_URL=postgresql://user:senha@localhost:5432/posweb
JWT_SECRET=troca-isso
JWT_ALGORITHM=HS256
JWT_EXPIRE_MINUTES=60
```
Gera o secret: `python -c "import secrets; print(secrets.token_urlsafe(32))"`

### 6. Adicionar CORS no `back/main.py`
Hoje o `main.py` só retorna "minha api". Falta o CORS pro front conseguir conversar:
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
Abre http://localhost:8000/docs — se o Swagger aparecer, **funcionou**. Marca a checklist do card e abre PR.

> [!tip] Falta o Alembic
> O repo ainda não tem Alembic (migrations). Vale adicionar (`pip install alembic` já vem no requirements, só rodar `alembic init`). Sem ele, as tabelas teriam que ser criadas na mão. Detalhe na [[Modelagem do Banco de Dados]].

---

## O que cada um faz depois do ambiente subir

### Modelagem (Fernando + Felipe)
1. Preenche `back/models/user_model.py` (hoje vazio) com o model `Usuario` — import `from db.database import Base`
2. Configura Alembic e gera primeira migration (ou cria as tabelas via `Base.metadata.create_all`)
3. **Avisa no grupo** que o Usuario tá pronto → libera o RF001
4. Faz os outros models em `back/models/` usando os **JSON do professor como base**: Aluno (`alunos.json`), Docente (`docentes.json`), Disciplina (`disciplinas.json`)

Passo a passo completo da modelagem na nota [[Modelagem do Banco de Dados]].

> [!tip] Não precisa inventar os campos
> O backend do professor já tem os dados. Abre `alunos.json`, `docentes.json`, `disciplinas.json` no https://github.com/DjanInfo/poswebserver e copia o shape. A gente só organiza melhor (separa nome/email, troca senha por senha_hash, etc.).

### RF001 Login (Milena)
Espera o Usuario ficar pronto. Aí (dentro de `back/`):
1. Cria `back/core/security.py` → `hash_password()`, `verify_password()` (passlib) e `create_access_token()` (python-jose)
2. Cria `back/schemas/auth.py` → `LoginRequest`, `TokenResponse`
3. Cria `back/routes/auth.py` → `POST /auth/login`
4. Inclui no `back/main.py`: `app.include_router(auth_router)`

Passo a passo completo, com código e testes, na nota [[RF001 - Login com JWT]].

### Cadastros (Nickolas / Hiago / Ana Clara F)
Esperam o RF001 (precisam do `hash_password()`). Depois fazem o mesmo padrão: schema Pydantic → rota POST em `back/routes/` → tratar duplicado (409).

---

## OBS

> [!warning] O que NÃO fazer
> - Comitar `.env` (vaza credencial)
> - Comitar `.venv/` (peso desnecessário)
> - Mexer direto na `main` — sempre branch + PR
> - Salvar senha em texto puro no banco
> - Mudar schema sem gerar migration no Alembic

> [!success] O que fazer
> - Mover card no Trello: `Sprint 1` → `em andamento` → `em revisão` → `concluído`
> - Sempre PR com revisão de outra pessoa
> - Reunião rápida toda segunda (15 min) só pra dizer o que cada um vai fazer na semana

---

## `.gitignore` recomendado pra pasta do projeto FastAPI

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

## Quando travar

Se alguém travar:
1. Manda no grupo do WhatsApp descrevendo o erro
2. Anexa o traceback inteiro (não só "deu erro")
3. Diz o que já tentou

Tá tudo amarrado em cima do Configurar Ambiente, então se ele atrasar muito atrasa todo mundo. Vamos focar nele primeiro.

---

## Links úteis

- [[00 - LEIA PRIMEIRO - Repositórios novos do professor]] — contexto da troca de repos
- Backend referência (Node): https://github.com/DjanInfo/poswebserver
- Frontend (React/Vite): https://github.com/DjanInfo/poswebreactdeploy
- Site no ar: https://pos-web-ifba.vercel.app/
- Figma: https://www.figma.com/design/QlqJVNBEj4SYcQanj9dv0D/Pos-Web
- FastAPI docs: https://fastapi.tiangolo.com/
- SQLAlchemy 2: https://docs.sqlalchemy.org/en/20/
- JWT no FastAPI: https://fastapi.tiangolo.com/tutorial/security/oauth2-jwt/
- Trello: https://trello.com/b/ewib1PdF/sistema-da-pos-graduacao-em-sistemas-web-back-end-utilizando-fastapi

---

Qualquer dúvida grita no grupo. Boa galera.
