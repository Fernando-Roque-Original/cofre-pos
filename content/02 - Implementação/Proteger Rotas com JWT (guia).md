---
title: Proteger Rotas com JWT (guia)
tags: [seguranca, jwt, middleware, autenticacao, sprint-2, backend, guia]
status: a-fazer
sprint: 2
card_trello: Proteger Rotas Privadas (Middleware JWT)
---

# Proteger Rotas com JWT (guia)

Guia da segurança: o conhecimento necessário, a explicação, e o roteiro pra cobrir os 4 itens do card.

> Status: a fazer. Esse card é a continuação do RF001 (login). O login GERA o token; aqui a gente EXIGE o token nas rotas privadas.

---

## O que o card pede (as 4 caixinhas)

1. Criar uma função de dependência (Depends) no FastAPI pra ler o cabeçalho Authorization
2. Decodificar e validar a assinatura e a expiração do token JWT
3. Extrair o ID e o Perfil (Role) do usuário que está no token
4. Retornar erro HTTP 401 caso o token seja inválido ou esteja ausente

---

## O conceito 

```
RF001 (login)  →  DÁ o token (prova "fiz login")
Proteção       →  EXIGE o token em toda rota privada (o porteiro)
```

 Foi criado um **porteiro** - uma função `get_current_user` - que toda rota privada usa. Sem token válido, a rota nem roda: devolve 401.

---

## A segurança explicada (o conhecimento necessário)

### Autenticação vs Autorização (não são a mesma coisa)
- **Autenticação** = "quem é você?" → é o **login**. Provar a identidade.
- **Autorização** = "o que você pode fazer?" → são as **permissões** (a role: admin/docente/estudante).

Este card é principalmente **autenticação** (tem um token válido?) e já **prepara** a autorização (extrai a role pra usar depois).

### O cabeçalho Authorization (o esquema "Bearer")
Depois do login, o cliente guarda o token e, em **toda requisição** a uma rota privada, manda no cabeçalho:
```
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
```
"Bearer" = "portador". É o padrão: quem **porta** o token tem acesso. Por isso o token tem que ser protegido (se vazar, qualquer um o usa até expirar).

### Como se valida um JWT
A função `jwt.decode(token, JWT_SECRET, algorithms=["HS256"])` faz duas checagens cruciais:
1. **Assinatura** → o token foi assinado com o NOSSO segredo? Se alguém adulterou o conteúdo (ex: mudou a role pra "admin"), a assinatura não bate → rejeita.
2. **Expiração (exp)** → o token ainda é válido no tempo? Se passou dos 60 min, rejeita.

Se qualquer uma falhar, o `jwt.decode` lança `JWTError`. A gente captura e devolve 401.

> Lembra: o JWT é **assinado, não criptografado** - qualquer um LÊ o conteúdo (jwt.io). O que protege é a assinatura: sem o JWT_SECRET, ninguém cria nem altera um token válido.

### O padrão "dependency" do FastAPI
No FastAPI, uma **dependency** é uma função que roda **antes** da rota. Você "pendura" ela na rota com `Depends(...)`. Se a dependency der erro (401), a rota **não executa**. É assim que a gente protege: a rota privada depende do `get_current_user`, que valida o token antes.

```python
@router.get("/me")
def quem_sou_eu(usuario = Depends(get_current_user)):  # roda get_current_user ANTES
    return usuario
```

---

## O roteiro (como cobrir as 4 caixinhas)

A gente constrói UMA função (`get_current_user`) que resolve as 4 de uma vez:

| # | Caixinha do card | Como o roteiro cobre |
|---|---|---|
| 1 | Função de dependência pra ler o Authorization | `HTTPBearer` lê o cabeçalho; a função recebe via `Depends` |
| 2 | Decodificar e validar assinatura + expiração | `jwt.decode(...)` dentro de try/except `JWTError` |
| 3 | Extrair ID e Role do token | `payload.get("sub")` e `payload.get("role")` |
| 4 | 401 se inválido ou ausente | `raise HTTPException(401)` no None e no JWTError |

---

## O código

### Passo 1 - A dependency (em `back/core/security.py`)

Imports novos no topo:
```python
from fastapi import Depends, HTTPException, status
from fastapi.security import HTTPBearer, HTTPAuthorizationCredentials
from jose import jwt, JWTError
```

A função no final do arquivo:
```python
security_scheme = HTTPBearer(auto_error=False)


def get_current_user(cred: HTTPAuthorizationCredentials = Depends(security_scheme)):
    erro_401 = HTTPException(
        status_code=status.HTTP_401_UNAUTHORIZED,
        detail="Token invalido ou ausente",
    )
    # [item 1 e 4] le o Authorization; se nao tiver token -> 401
    if cred is None:
        raise erro_401
    # [item 2] decodifica e valida assinatura + expiracao
    try:
        payload = jwt.decode(cred.credentials, JWT_SECRET, algorithms=[JWT_ALGORITHM])
    except JWTError:
        raise erro_401
    # [item 3] extrai id e role
    user_id = payload.get("sub")
    role = payload.get("role")
    if user_id is None:
        raise erro_401
    return {"id": user_id, "role": role}
```

- `HTTPBearer(auto_error=False)` → lê o `Authorization: Bearer <token>`. O `auto_error=False` deixa a gente tratar o "ausente" pra devolver **401** (o padrão seria 403).

### Passo 2 - Uma rota de teste protegida

Pra ver funcionando, cria uma rota que exige o token. Pode ser num arquivo novo `back/routes/me_routes.py` (ou na auth_routes):
```python
from fastapi import APIRouter, Depends
from core.security import get_current_user

router = APIRouter(tags=["Eu"])


@router.get("/me")
def quem_sou_eu(usuario: dict = Depends(get_current_user)):
    return usuario
```
E liga no `main.py` (`include_router`).

### Passo 3 - Proteger rotas de verdade
Pra proteger qualquer rota existente, é só adicionar o `Depends(get_current_user)`:
```python
@router.get("/")
def listar_alunos(db: Session = Depends(get_db), usuario = Depends(get_current_user)):
    ...
```
Sem token válido, a listagem nem roda.

---

## Como testar (no Swagger)

1. `uvicorn main:app --reload` -> `/docs`
2. **Sem token:** chama `GET /me` direto -> deve dar **401**
3. **Pega um token:** chama `POST /login` (admin@ifba.edu.br / admin123) -> copia o `access_token`
4. **Autoriza:** clica no botão **"Authorize"** (cadeado, topo do Swagger) -> cola o token -> Authorize
5. **Com token:** chama `GET /me` de novo -> agora devolve `{ id, role }` (200)
6. **Token inválido:** muda uma letra do token -> **401**

---

## Bonus: proteger por PERFIL (autorização)

Como a gente extrai a `role`, dá pra fazer rotas "só de admin". Exemplo de uma dependency extra:
```python
def somente_admin(usuario: dict = Depends(get_current_user)):
    if usuario["role"] != "admin":
        raise HTTPException(status_code=403, detail="Acesso negado")
    return usuario
```
Aí uma rota com `Depends(somente_admin)` só deixa admin entrar. (403 = autenticado, mas sem permissao; diferente de 401 = não autenticado)

---

## Cuidados de segurança

- O token vai no cabeçalho **Authorization**, nunca na URL (a URL fica em logs/histórico)
- 401 = não autenticado (sem token / token ruim). 403 = autenticado, mas sem permissão. Não confunda.
- O frontend guarda o token; se vazar, vale até expirar - por isso a expiração curta (60 min)
- Em produção, tudo por HTTPS (senão o token trafega aberto)

---

## Links

- [[RF001 - Login com JWT]] - o login que gera o token
- [[RF001 - Registro de implementação (o que fizemos)]] - como o token é gerado
- [[Entendendo FastAPI e CORS]] - base do FastAPI (e o que é Depends)
- [[Conceitos Python do Código (def, return, etc)]] - os termos do código
- Card no Trello: Proteger Rotas Privadas (Middleware JWT)
