---
title: RF001 - Registro de implementação
tags: [rf001, login, jwt, registro, entrega, grupo-5]
status: concluído
sprint: 1
---

# RF001 — Como implementamos o Login (registro)

> **Status: 100% concluído** (os 5 itens do checklist do Trello).

Documento do que foi **criado, mudado e adicionado** no código pra fazer o login funcionar — e **por quê**. Serve pra explicar a entrega pro grupo e pro professor.

---

## O que o RF001 pedia

Uma rota **`POST /login`** que: recebe email + senha, valida, e devolve um **token JWT**.

> **Importante (escopo):** o RF001 é só o login que **dá** o token. **Usar** esse token pra proteger outras rotas é um card **separado** ("Proteger Rotas Privadas"), da **Sprint 2**.

---

## O fluxo que construímos

```
1. Cliente manda:  POST /login  { email, senha }
        ↓
2. Pydantic valida o formato (LoginRequest)
        ↓
3. Busca o usuário no banco pelo email (SQLAlchemy)
        ↓
4. Compara a senha digitada com o hash do banco (bcrypt)
   ├─ não bate / não existe → 401 "Credenciais inválidas"
   └─ bate → segue
        ↓
5. Gera o token JWT com { id, role, validade }
        ↓
6. Devolve { access_token, token_type } com status 200
```

---

## Arquivos — o que criamos e mudamos

| Arquivo | Novo ou mudado | Papel |
|---|---|---|
| `back/core/security.py` | novo | hash de senha + geração do JWT |
| `back/schemas/auth_schema.py` | novo | o "contrato" (o que entra e sai) |
| `back/routes/auth_routes.py` | novo | a rota `POST /login` |
| `back/main.py` | mudado | ligou a rota de auth na aplicação |
| `back/seed_usuario.py` | novo | cria um usuário de teste no banco |
| `requirements.txt` | corrigido | bugs de versão (jose e bcrypt) |

---

## Cada arquivo, explicado (o que tem e por que)

### `back/core/security.py`
Três funções:
- `hash_password(senha)` → transforma a senha em hash bcrypt (usado no cadastro/seed)
- `verify_password(senha, hash)` → confere a senha no login (devolve True/False)
- `create_access_token(id, role)` → gera o JWT com o id, a role e a validade

**Por que num arquivo separado:** essas funções são **reutilizadas** (o cadastro também faz hash, o login faz verify). Concentrar num lugar evita repetir código.

### `back/schemas/auth_schema.py`
- `LoginRequest` → o que o cliente **manda** (`email`, `senha`)
- `TokenResponse` → o que a API **devolve** (`access_token`, `token_type`)

**Por que:** o Pydantic usa esses schemas pra **validar** a entrada e **formatar** a saída. É o contrato da rota.

### `back/routes/auth_routes.py`
A rota `POST /login`. Faz a busca do usuário, a verificação da senha, a geração do token e o tratamento do erro 401.

**Por que separado do main:** organização — cada grupo de rotas (auth, alunos...) num arquivo. O `main.py` só **liga** elas.

### `back/main.py` (mudança)
Adicionamos:
```python
from routes import aluno_routes, auth_routes
...
app.include_router(auth_routes.router)
```
**Por que:** criar a rota num arquivo não basta — tem que **registrar** ela na aplicação com `include_router`, senão o FastAPI não a conhece.

### `back/seed_usuario.py`
Cria um usuário de teste: `admin@ifba.edu.br` / `admin123` (com a senha já em hash).

**Por que:** pra testar o login, precisa existir **pelo menos um usuário** no banco. O seed cria ele com o hash certo.

### `requirements.txt` (correções)
- `python-jose[cryptography]=3.3.0` → **`==`** (tinha um `=` só, sintaxe errada)
- Adicionado **`bcrypt==4.0.1`** (a versão nova do bcrypt era incompatível com o passlib)

**Por que:** sem essas correções, o hash quebrava. Detalhe nos problemas abaixo.

---

## Os 5 itens do checklist → onde estão no código

| # | Item do Trello | Onde |
|---|---|---|
| 1 | Criar rota POST /login | `auth_routes.py` → `@router.post("/login")` |
| 2 | Buscar usuário por email + comparar hash | `db.query(Usuario).filter(email...)` + `verify_password()` |
| 3 | Gerar JWT com ID + Role | `create_access_token(user_id=usuario.id, role=usuario.role)` |
| 4 | Retornar token com HTTP 200 | `return TokenResponse(...)` (200 é o padrão) |
| 5 | Tratar 401 pra credenciais inválidas | `raise HTTPException(401, "Credenciais inválidas")` |

---

## Problemas que enfrentamos (e como resolvemos)

### 1. Hash quebrava — incompatibilidade passlib × bcrypt
**Erro:** `module 'bcrypt' has no attribute '__about__'` / `password cannot be longer than 72 bytes`.
**Causa:** o `passlib 1.7.4` (de 2020) tenta ler uma propriedade que o `bcrypt 4.1+` removeu.
**Solução:** fixar `bcrypt==4.0.1` (última versão compatível) no requirements e instalar.

### 2. Login dava 500 no Swagger
**Causa:** o servidor uvicorn estava rodando com o bcrypt antigo na memória — o `pip install` do fix não reinicia o servidor sozinho.
**Solução:** parar (`Ctrl+C`) e subir o uvicorn de novo.
**Lição:** erro **500** = problema no servidor; o traceback real aparece no **terminal do uvicorn**, não no navegador.

---

## Como testar (reproduzir)

1. Garante um usuário no banco: `python seed_usuario.py`
2. Sobe o servidor: `uvicorn main:app --reload` (de dentro de `back/`)
3. Abre `http://localhost:8000/docs` → seção **Autenticação** → `POST /login`
4. **Certo:** `{ "email": "admin@ifba.edu.br", "senha": "admin123" }` → **200** + token
5. **Errado:** senha errada → **401** "Credenciais inválidas"
6. Cola o token em https://jwt.io → vê `sub` (id), `role`, `exp`

---

## O que é RF001 e o que NÃO é (escopo)

| | |
|---|---|
| **É RF001** | o login que **gera e devolve** o token |
| **NÃO é RF001** (é Sprint 2) | **usar** o token pra **proteger** rotas (o `get_current_user`, exigir login pra acessar um endpoint) |

Por isso o RF001 está **completo** mesmo sem proteção de rotas — a proteção é a continuação, no card "Proteger Rotas Privadas".

---

## Links

- [[RF001 - Login com JWT]] — o guia/passo a passo original
- [[Conceitos Python do Código (def, return, etc)]] — o que cada termo do código significa
- [[Entendendo FastAPI e CORS]] — base do FastAPI
- Card no Trello: [RF001] Autenticação e JWT (POST /login)
