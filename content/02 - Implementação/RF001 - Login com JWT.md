---
title: RF001 - Login com JWT
tags: [rf001, jwt, auth, sprint-1, backend]
card_trello: https://trello.com/c/Pj2AGibi
prazo: 31/mai
---

# RF001 — Login com JWT

Guia completo: do que estudar até deploy em produção. Lê tudo antes de codar — vai poupar horas de retrabalho.

> [!info] Card no Trello
> [RF001] Autenticação e JWT (POST /login) — Sprint 1, prazo 31/mai

> [!important] Contexto novo — lê a [[00 - LEIA PRIMEIRO - Repositórios novos do professor]]
> O backend de referência do professor (https://github.com/DjanInfo/poswebserver) **não tem login nenhum** e guarda senha em texto puro. O RF001 é o nosso maior diferencial: a gente entrega o que falta no sistema dele. O modelo de usuário abaixo é baseado no `usuarios.json` dele, mas feito direito.

> [!danger] AJUSTE OS CAMINHOS — este guia foi escrito com `app/`, mas o repo do Hiago usa `back/`
> Onde você ler `app.` nos exemplos, troca por isto (estrutura real do `hiag-code/poswebreactdeploy`):
>
> | No guia (exemplos) | No repo do Hiago (use isto) |
> |---|---|
> | `app/main.py` | `back/main.py` |
> | `app.database` | `db.database` |
> | `app.models.usuario` | `models.user_model` |
> | `app.routers.auth` | `routes.auth` |
> | `app.core.security` | `core.security` |
> | `app.config` | usa `os.getenv` ou um `config.py` em `back/` |
> | `uvicorn app.main:app` | `cd back` e `uvicorn main:app` |
>
> **Atalho importante:** o `get_db` **já existe** em `back/db/database.py`. **Pule o Passo 6** (não crie outro) — só importe: `from db.database import get_db`. O `Base` e o `SessionLocal` também já estão lá.

---

## 0. O que estudar antes

Se você nunca mexeu com autenticação, gasta 1–2h estudando esses conceitos. Vai render 10x mais na hora de codar.

### Conceitos essenciais

| Tópico | Por que precisa | Onde estudar |
|---|---|---|
| **Hash vs criptografia** | Hash é mão única (não dá pra reverter), criptografia tem chave. Senha SEMPRE usa hash. | https://www.welivesecurity.com/br/2018/01/24/hash-criptografia/ |
| **bcrypt** | Algoritmo de hash lento de propósito — proteção contra força bruta. | https://en.wikipedia.org/wiki/Bcrypt |
| **JWT (JSON Web Token)** | Token assinado que carrega informação. Stateless: o servidor não guarda nada. | https://jwt.io/introduction |
| **HTTP status codes** | 200 OK, 401 Unauthorized, 422 Unprocessable Entity — fundamentais pra API REST. | https://developer.mozilla.org/pt-BR/docs/Web/HTTP/Status |
| **Stateless vs Stateful** | JWT é stateless. Sessão tradicional é stateful. Cada um com sua vantagem. | Pesquisa "JWT vs session" |
| **OAuth2 Password Flow** | Padrão que o FastAPI usa por baixo. Não precisa dominar, mas entenda o básico. | https://fastapi.tiangolo.com/tutorial/security/ |

### Bibliotecas que você vai usar

| Lib                   | O que faz                             | Doc oficial                                                  |
| --------------------- | ------------------------------------- | ------------------------------------------------------------ |
| **FastAPI**           | Cria a rota `/login` e gera o Swagger | https://fastapi.tiangolo.com/                                |
| **Pydantic v2**       | Valida entrada/saída da API           | https://docs.pydantic.dev/latest/                            |
| **SQLAlchemy 2**      | Conversa com o banco (ORM)            | https://docs.sqlalchemy.org/en/20/                           |
| **passlib[bcrypt]**   | Hash e verificação de senha           | https://passlib.readthedocs.io/                              |
| **python-jose**       | Gera e valida JWT                     | https://python-jose.readthedocs.io/                          |
| **pydantic-settings** | Carrega configurações do `.env`       | https://docs.pydantic.dev/latest/concepts/pydantic_settings/ |
| **pytest + httpx**    | Testes automatizados da API           | https://fastapi.tiangolo.com/tutorial/testing/               |

### Vídeos curtos (se preferir vídeo)

- *FastAPI Authentication with JWT (Bitfumes)* — ~30 min, mostra do zero em inglês
- *Curso intensivo Python FastAPI* — qualquer playlist BR no YouTube com "FastAPI JWT"
- *JWT explicado em 5 minutos* — busca "JWT explained" no YouTube

### Anatomia de um JWT (entenda antes de gerar um)

Um JWT tem 3 partes separadas por ponto:

```
eyJhbGciOiJIUzI1NiJ9 . eyJzdWIiOiIxIiwicm9sZSI6ImFkbWluIn0 . k7c8H...
       HEADER                    PAYLOAD                    ASSINATURA
```

| Parte | Conteúdo | Codificação |
|---|---|---|
| **Header** | `{ "alg": "HS256", "typ": "JWT" }` | Base64 (NÃO criptografado) |
| **Payload** | `{ "sub": "1", "role": "admin", "exp": 123456 }` | Base64 (NÃO criptografado) |
| **Assinatura** | hash(header + payload + JWT_SECRET) | Hash HMAC |

> [!warning] JWT não é criptografado, é assinado
> Qualquer pessoa consegue ler o payload do JWT colando em https://jwt.io.
> O que protege é a **assinatura** — sem o `JWT_SECRET`, ninguém consegue criar um token válido novo.
> Por isso: **nunca coloque dado sensível no payload** (senha, CPF, etc).

---

## 1. Antes de começar — confere se isso existe

Não adianta começar o login se essas coisas não estão prontas. Pergunta no grupo:

- [x] O ambiente do projeto tá rodando? (`uvicorn app.main:app --reload` sobe sem erro)
- [x] Tem `requirements.txt` com `python-jose[cryptography]` e `passlib[bcrypt]`?
- [x] O model `Usuario` foi criado pela Pessoa 2?
- [x] A migration do Alembic foi aplicada? (tabela `usuarios` existe no banco)
- [ ] Tem pelo menos 1 usuário cadastrado no banco pra você testar?

Se faltar qualquer um, cobra antes de começar. Sem isso você vai travar.

---

## 2. O que precisa no banco de dados

A tabela `usuarios` é a única coisa que o login toca. Estrutura:

| Coluna | Tipo | Pra quê |
|---|---|---|
| `id` | int, PK auto | Identificador único, vai no JWT |
| `email` | varchar(255), único | Login do usuário |
| `senha_hash` | varchar(255) | Senha já com hash bcrypt (NUNCA texto puro) |
| `role` | varchar(20) | `estudante`, `docente` ou `admin` (= o `perfil` do prof) |
| `ativo` | boolean, default true | Pra bloquear sem deletar |
| `criado_em` | timestamp | Auditoria |

> [!note] De onde veio esse modelo
> No `usuarios.json` do professor o usuário é `{ id, login, senha, perfil }` — com a senha em texto puro e o campo `login` guardando o nome. A gente arruma: separa `email`, troca por `senha_hash`, e o `perfil` dele (ex: "Estudante") vira nosso `role`. Mesmos perfis dele, só que seguro.

Model em SQLAlchemy (vai estar em `app/models/usuario.py`):

```python
from sqlalchemy import Column, Integer, String, Boolean, DateTime
from sqlalchemy.sql import func
from app.database import Base

class Usuario(Base):
    __tablename__ = "usuarios"

    id = Column(Integer, primary_key=True, index=True)
    email = Column(String(255), unique=True, nullable=False, index=True)
    senha_hash = Column(String(255), nullable=False)
    role = Column(String(20), nullable=False)
    ativo = Column(Boolean, default=True)
    criado_em = Column(DateTime, server_default=func.now())
```

> [!warning] Antes da migration
> Se a Pessoa 2 ainda não fez essa tabela, você não tem onde gravar/buscar nada. Combina com ela pra fazer ESSE model primeiro.

---

## 3. Como popular pra testar

Cria `seed.py` na raiz do backend:

```python
# pos-web-backend/seed.py
from app.database import SessionLocal
from app.models.usuario import Usuario
from app.core.security import hash_password

db = SessionLocal()

if not db.query(Usuario).filter_by(email="admin@ifba.edu.br").first():
    admin = Usuario(
        email="admin@ifba.edu.br",
        senha_hash=hash_password("admin123"),
        role="admin"
    )
    db.add(admin)
    db.commit()
    print("Admin criado")
else:
    print("Admin já existe")

db.close()
```

Roda uma vez: `python seed.py`

---

## 4. Fluxo do login (entenda antes de codar)

```mermaid
sequenceDiagram
    participant C as Cliente
    participant API as FastAPI
    participant DB as PostgreSQL
    participant JWT as python-jose

    C->>API: POST /auth/login<br/>{ email, senha }
    API->>API: Pydantic valida formato
    API->>DB: SELECT * FROM usuarios<br/>WHERE email = ?
    DB-->>API: usuario (ou None)

    alt usuário não existe OU senha errada
        API-->>C: 401 Unauthorized
    else credenciais OK
        API->>JWT: encode({ sub, role, exp })
        JWT-->>API: token assinado
        API-->>C: 200 OK { access_token }
    end
```

---

## 5. Criação passo a passo (em 12 micro-passos)

Faz **um por vez**, testa antes de passar pro próximo. Não escreve tudo de uma vez.

### Passo 1 — Criar a estrutura de pastas

Dentro de `pos-web-backend/app/`:

```bash
mkdir core schemas routers
touch core/__init__.py core/security.py
touch schemas/__init__.py schemas/auth.py
touch routers/__init__.py routers/auth.py
touch dependencies.py
```

(No Windows usa `type nul > arquivo.py` em vez de `touch`.)

### Passo 2 — Criar `config.py` com as variáveis do `.env`

`app/config.py`:

```python
from pydantic_settings import BaseSettings


class Settings(BaseSettings):
    DATABASE_URL: str
    JWT_SECRET: str
    JWT_ALGORITHM: str = "HS256"
    JWT_EXPIRE_MINUTES: int = 60

    class Config:
        env_file = ".env"


settings = Settings()
```

Testa importando no Python:
```bash
python -c "from app.config import settings; print(settings.JWT_ALGORITHM)"
```
Deve imprimir `HS256`.

### Passo 3 — Função de hash

`app/core/security.py`:

```python
from passlib.context import CryptContext

pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")


def hash_password(senha_pura: str) -> str:
    return pwd_context.hash(senha_pura)


def verify_password(senha_pura: str, senha_hash: str) -> bool:
    return pwd_context.verify(senha_pura, senha_hash)
```

Testa no Python:
```bash
python -c "from app.core.security import hash_password, verify_password; h = hash_password('123'); print(h); print(verify_password('123', h)); print(verify_password('456', h))"
```
Deve imprimir o hash, `True`, `False`.

### Passo 4 — Função de gerar JWT

Adiciona em `app/core/security.py`:

```python
from datetime import datetime, timedelta, timezone
from jose import jwt
from app.config import settings


def create_access_token(user_id: int, role: str) -> str:
    expire = datetime.now(timezone.utc) + timedelta(
        minutes=settings.JWT_EXPIRE_MINUTES
    )
    payload = {
        "sub": str(user_id),
        "role": role,
        "exp": expire,
    }
    return jwt.encode(payload, settings.JWT_SECRET, algorithm=settings.JWT_ALGORITHM)
```

Testa:
```bash
python -c "from app.core.security import create_access_token; print(create_access_token(1, 'admin'))"
```
Deve imprimir um JWT. Cola em https://jwt.io e confere que `sub=1, role=admin, exp=...`.

### Passo 5 — Schemas Pydantic

`app/schemas/auth.py`:

```python
from pydantic import BaseModel, EmailStr


class LoginRequest(BaseModel):
    email: EmailStr
    senha: str


class TokenResponse(BaseModel):
    access_token: str
    token_type: str = "bearer"
```

### Passo 6 — Dependência `get_db`

`app/dependencies.py`:

```python
from app.database import SessionLocal


def get_db():
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()
```

### Passo 7 — A rota `/auth/login`

`app/routers/auth.py`:

```python
from fastapi import APIRouter, Depends, HTTPException, status
from sqlalchemy.orm import Session

from app.dependencies import get_db
from app.models.usuario import Usuario
from app.schemas.auth import LoginRequest, TokenResponse
from app.core.security import verify_password, create_access_token

router = APIRouter(prefix="/auth", tags=["Autenticação"])


@router.post("/login", response_model=TokenResponse)
def login(payload: LoginRequest, db: Session = Depends(get_db)):
    usuario = db.query(Usuario).filter(Usuario.email == payload.email).first()

    if not usuario or not verify_password(payload.senha, usuario.senha_hash):
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Credenciais inválidas",
        )

    if not usuario.ativo:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Usuário desativado",
        )

    token = create_access_token(user_id=usuario.id, role=usuario.role)
    return TokenResponse(access_token=token)
```

### Passo 8 — Incluir o router no `main.py`

Adiciona no `app/main.py`:

```python
from app.routers import auth

app.include_router(auth.router)
```

### Passo 9 — Subir o servidor

```bash
uvicorn app.main:app --reload
```

Abre http://localhost:8000/docs — o endpoint `POST /auth/login` deve aparecer.

### Passo 10 — Rodar o seed

```bash
python seed.py
```

### Passo 11 — Testar manualmente no Swagger

Vê seção 6 abaixo.

### Passo 12 — Escrever os testes automatizados

Vê seção 7 abaixo.

---

## 6. Como testar — manual

### Pelo Swagger (rápido)

1. Servidor rodando: `uvicorn app.main:app --reload`
2. Abre `http://localhost:8000/docs`
3. Expande `POST /auth/login` → "Try it out"
4. Cola:
   ```json
   { "email": "admin@ifba.edu.br", "senha": "admin123" }
   ```
5. Clica "Execute"

Resposta esperada (200):
```json
{
  "access_token": "eyJhbGciOiJIUzI1NiIs...",
  "token_type": "bearer"
}
```

### Pelo curl/Postman/Insomnia

```bash
curl -X POST http://localhost:8000/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"admin@ifba.edu.br","senha":"admin123"}'
```

### Matriz de cenários manuais (testa TODOS antes do PR)

| # | Cenário | Entrada | Status | Detail |
|---|---|---|---|---|
| 1 | Login OK | email + senha certos | 200 | token JWT |
| 2 | Email não existe | `noexist@x.com` + qualquer | 401 | "Credenciais inválidas" |
| 3 | Senha errada | email certo + senha errada | 401 | "Credenciais inválidas" |
| 4 | Email mal formado | `"naoehemail"` | 422 | Pydantic detalha o erro |
| 5 | Sem email | `{"senha":"x"}` | 422 | "field required" |
| 6 | Sem senha | `{"email":"a@b.c"}` | 422 | "field required" |
| 7 | Usuário desativado | email com `ativo=False` | 401 | "Usuário desativado" |
| 8 | Body vazio | `{}` | 422 | dois "field required" |

### Validar o JWT gerado

1. Copia o `access_token` da resposta
2. Cola em https://jwt.io
3. Deve mostrar:
   ```json
   {
     "sub": "1",
     "role": "admin",
     "exp": 1735689600
   }
   ```

> [!warning] Se a página jwt.io mostrar "Invalid Signature"
> É normal — você não colou o JWT_SECRET lá. O que importa é o payload ser legível e os campos certos.

---

## 7. Como testar — automatizado (pytest)

### Por que automatizado vale a pena

- Roda em 3 segundos
- Não esquece de nenhum cenário
- Quem revisar seu PR consegue ver que tá funcionando
- Quando alguém quebrar o login no futuro, o teste avisa

### Instala as dependências de teste

Adiciona no `requirements.txt`:

```
pytest==8.3.3
httpx==0.27.2
pytest-asyncio==0.24.0
```

```bash
pip install -r requirements.txt
```

### Cria `tests/test_auth.py`

```python
import pytest
from fastapi.testclient import TestClient
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker
from sqlalchemy.pool import StaticPool

from app.main import app
from app.database import Base
from app.dependencies import get_db
from app.models.usuario import Usuario
from app.core.security import hash_password


# banco em memória só pros testes
engine = create_engine(
    "sqlite:///:memory:",
    connect_args={"check_same_thread": False},
    poolclass=StaticPool,
)
TestingSessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)


def override_get_db():
    try:
        db = TestingSessionLocal()
        yield db
    finally:
        db.close()


app.dependency_overrides[get_db] = override_get_db


@pytest.fixture(autouse=True)
def setup_database():
    Base.metadata.create_all(bind=engine)
    db = TestingSessionLocal()
    db.add(Usuario(
        email="admin@ifba.edu.br",
        senha_hash=hash_password("admin123"),
        role="admin",
        ativo=True,
    ))
    db.add(Usuario(
        email="banido@ifba.edu.br",
        senha_hash=hash_password("123"),
        role="aluno",
        ativo=False,
    ))
    db.commit()
    db.close()
    yield
    Base.metadata.drop_all(bind=engine)


client = TestClient(app)


def test_login_sucesso():
    r = client.post("/auth/login", json={
        "email": "admin@ifba.edu.br",
        "senha": "admin123",
    })
    assert r.status_code == 200
    body = r.json()
    assert "access_token" in body
    assert body["token_type"] == "bearer"


def test_login_email_nao_existe():
    r = client.post("/auth/login", json={
        "email": "ninguem@x.com",
        "senha": "qualquer",
    })
    assert r.status_code == 401
    assert r.json()["detail"] == "Credenciais inválidas"


def test_login_senha_errada():
    r = client.post("/auth/login", json={
        "email": "admin@ifba.edu.br",
        "senha": "errada",
    })
    assert r.status_code == 401
    assert r.json()["detail"] == "Credenciais inválidas"


def test_login_email_invalido():
    r = client.post("/auth/login", json={
        "email": "naoehemail",
        "senha": "x",
    })
    assert r.status_code == 422


def test_login_usuario_desativado():
    r = client.post("/auth/login", json={
        "email": "banido@ifba.edu.br",
        "senha": "123",
    })
    assert r.status_code == 401
    assert r.json()["detail"] == "Usuário desativado"


def test_login_body_vazio():
    r = client.post("/auth/login", json={})
    assert r.status_code == 422
```

### Roda os testes

```bash
pytest tests/ -v
```

Saída esperada:
```
test_auth.py::test_login_sucesso PASSED
test_auth.py::test_login_email_nao_existe PASSED
test_auth.py::test_login_senha_errada PASSED
test_auth.py::test_login_email_invalido PASSED
test_auth.py::test_login_usuario_desativado PASSED
test_auth.py::test_login_body_vazio PASSED

============= 6 passed in 0.45s =============
```

Se todos passarem, pode abrir PR.

---

## 8. Segurança — explicação profunda

Cada decisão técnica aqui tem motivo de segurança. Entenda **por que** antes de copiar e colar.

### 8.1 Por que bcrypt e não SHA-256 ou MD5?

| Algoritmo | Tempo por hash | Usa pra senha? |
|---|---|---|
| MD5 | ~0.001ms | NÃO — quebrado, colisões conhecidas |
| SHA-256 | ~0.01ms | NÃO — rápido demais, vulnerável a brute force |
| bcrypt (rounds=12) | ~300ms | SIM — lento de propósito |
| Argon2 | ~300ms | SIM — mais novo, também aceito |

bcrypt é **lento de propósito**. Um atacante que conseguir o dump do banco vai levar ~300ms por tentativa em vez de microssegundos. Brute force vira inviável.

**Custo (rounds):** o `CryptContext(schemes=["bcrypt"])` usa 12 rounds por padrão. Não diminui.

### 8.2 Por que `verify_password` em vez de `==`?

```python
# ERRADO — timing attack
if senha_digitada == senha_hash_do_banco:

# CERTO
if pwd_context.verify(senha_pura, senha_hash):
```

**Timing attack:** Python compara strings caractere por caractere e para no primeiro diferente. Atacante mede o tempo de resposta e descobre quantos caracteres acertou. `verify` usa **comparação em tempo constante** — não vaza essa informação.

### 8.3 Por que mesma mensagem pra "email errado" e "senha errada"?

```python
# ERRADO — enumeração de usuários
if not usuario:
    raise HTTPException(401, "Email não cadastrado")
if not verify_password(...):
    raise HTTPException(401, "Senha incorreta")

# CERTO
if not usuario or not verify_password(...):
    raise HTTPException(401, "Credenciais inválidas")
```

Com mensagens diferentes, o atacante consegue **descobrir emails válidos** testando milhares. Depois ataca só os que existem. Com a mesma mensagem, ele não sabe se errou email ou senha.

### 8.4 Por que JWT_SECRET no `.env`?

```python
# ERRADO
SECRET = "minha-chave"   # vai pro Git → vaza no GitHub

# CERTO
JWT_SECRET no .env (gitignore'd), produção tem secret diferente do dev
```

Já houve casos de empresas com secret no Git público e crackers gerarem tokens válidos pra qualquer usuário.

### 8.5 Por que `exp` no JWT?

JWT é **stateless** — uma vez emitido, o servidor não tem como invalidar (a não ser com blocklist, que é complexo). Solução: o token **expira sozinho**. Padrão sugerido:

| Tipo | Tempo |
|---|---|
| Token de produção sensível (banco) | 15 min |
| Token de aplicação comum | 60 min |
| Refresh token (não vamos usar) | 7 dias |

Sem `exp`, token vazado = acesso vitalício. Crítico.

### 8.6 Por que não salvar dado sensível no JWT?

JWT **NÃO é criptografado, é assinado**. Qualquer pessoa cola em jwt.io e lê. Então:

```python
# RUIM
payload = { "sub": 1, "cpf": "12345678900", "senha": "..." }

# OK
payload = { "sub": "1", "role": "admin", "exp": ... }
```

Só ID + role + exp. Nada que seja segredo.

### 8.7 Por que `sub` como string?

A spec do JWT (RFC 7519) diz que `sub` deve ser string. Alguns validadores (incluindo python-jose em versões futuras) vão rejeitar se for inteiro. Sempre `str(user_id)`.

### 8.8 Por que checar `ativo`?

Cenário: aluno cola sua senha no Discord do curso. Você desativa o usuário. Se o login não checar `ativo`, ele continua entrando. Sempre checa.

### 8.9 O que NÃO está protegido por esse card (e como complementar depois)

| Risco | Proteção | Onde implementar |
|---|---|---|
| Brute force no login | Rate limiting (ex: máx 5 tentativas/min por IP) | Sprint 3 — slowapi |
| Token roubado em XSS | HttpOnly cookie em vez de localStorage | Decisão de frontend |
| Token roubado em MITM | HTTPS obrigatório em produção | Deploy (Railway/Render já forçam) |
| Logs de auditoria | Registrar tentativas falhadas | Card "[NF006] Middleware de Logs" |
| Refresh token | Renovação sem novo login | Não está no escopo |

### 8.10 Checklist de segurança (cobre antes do PR)

- [ ] Senha sempre salva como hash bcrypt
- [ ] Comparação com `verify_password` (não `==`)
- [ ] Mesma mensagem de erro pra email errado e senha errada
- [ ] `JWT_SECRET` no `.env`, `.env` no `.gitignore`
- [ ] `JWT_SECRET` tem 32+ caracteres aleatórios
- [ ] Token tem `exp` (60min)
- [ ] Payload do JWT só tem `sub`, `role`, `exp`
- [ ] Checa `ativo == True` antes de gerar token
- [ ] `sub` é string, não int
- [ ] Nenhuma senha em texto puro em log, response ou banco

---

## 9. Variáveis de ambiente

`.env` (não comita):

```env
DATABASE_URL=postgresql://user:senha@localhost:5432/posweb
JWT_SECRET=cole-aqui-um-secret-aleatorio-de-32-chars
JWT_ALGORITHM=HS256
JWT_EXPIRE_MINUTES=60
```

Gera o secret:
```bash
python -c "import secrets; print(secrets.token_urlsafe(32))"
```

`.env.example` (comita) com valores fake.

---

## 10. O que NÃO faz parte desse card

- Proteger outras rotas com JWT → card *"Proteger Rotas Privadas"* (Sprint 2)
- Refresh token → não tá no escopo
- Logout no servidor → JWT é stateless, frontend só apaga o token
- Recuperação de senha → não pedido
- Cadastro de usuário → RF002/RF003

Foca SÓ no `/login` aqui.

---

## 11. Definição de "pronto"

Antes de mover o card pra **em revisão**:

- [ ] Os 5 itens da checklist do Trello marcados
- [ ] Os 8 cenários manuais da seção 6 passam
- [ ] Os 6 testes automatizados passam (`pytest tests/ -v`)
- [ ] Checklist de segurança da seção 8.10 toda marcada
- [ ] Swagger mostra o endpoint funcionando
- [ ] `.env` no `.gitignore`, `.env.example` no repo
- [ ] Commit message clara: `feat(auth): implementa login com JWT`
- [ ] PR aberto pedindo review
- [ ] Pelo menos 1 colega aprovou

---

## 12. Comandos rápidos

```bash
# subir o servidor
uvicorn app.main:app --reload

# rodar testes
pytest tests/ -v

# popular banco de teste
python seed.py

# gerar JWT_SECRET
python -c "import secrets; print(secrets.token_urlsafe(32))"

# fluxo de PR
git checkout -b feature/rf001-login-jwt
git add app/core/ app/schemas/auth.py app/routers/auth.py app/dependencies.py app/main.py tests/
git commit -m "feat(auth): implementa login com JWT"
git push origin feature/rf001-login-jwt
```

---

## 13. Quando travar

Erro comum + onde olhar:

| Erro | Provável causa |
|---|---|
| `ModuleNotFoundError: No module named 'app'` | Rodar de fora da pasta `pos-web-backend/` |
| `sqlalchemy.exc.OperationalError: connection refused` | Postgres não tá rodando ou DATABASE_URL errada |
| `ValidationError: email field required` | Payload não tem o campo certo (atenção: é `email` e `senha`, sem acento) |
| `JWTError: Signature verification failed` | JWT_SECRET diferente entre quem gerou e quem está validando |
| `401 Unauthorized` em tudo | Roda o seed; ou senha errada mesmo |
| `passlib hash crashou no Windows` | Confere se instalou `passlib[bcrypt]` (com colchetes) |

Joga o traceback inteiro no grupo. Não joga só "deu erro".

---

## Links

- Card no Trello: [RF001] Autenticação e JWT
- python-jose: https://python-jose.readthedocs.io/
- passlib bcrypt: https://passlib.readthedocs.io/en/stable/lib/passlib.hash.bcrypt.html
- JWT no FastAPI: https://fastapi.tiangolo.com/tutorial/security/oauth2-jwt/
- Testando FastAPI: https://fastapi.tiangolo.com/tutorial/testing/
- jwt.io decoder: https://jwt.io
- OWASP Authentication Cheat Sheet: https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html
- Notas relacionadas: [[00 - LEIA PRIMEIRO - Repositórios novos do professor]] · [[Modelagem do Banco de Dados]] · [[Como começar - Sprint 1]] · [[Plano de Implementação - Desafio 5]]

---

Travou em qualquer passo, joga o erro inteiro no grupo. Boa.
