---
title: Proteger Rotas - Registro de implementação
tags: [seguranca, jwt, middleware, autenticacao, registro, sprint-2, backend]
status: concluído
sprint: 2
card_trello: Proteger Rotas Privadas (Middleware JWT)
---

# Proteger Rotas - Registro de implementação (o que fizemos)

> Status: concluído (os 4 itens do checklist).
> Testado: GET /me sem token -> 401; com token -> 200 com os dados do usuário.

Documento de tudo que usamos, como funciona, e por que cada coisa está em cada lugar.

---

## O que o card pedia

1. Criar uma função de dependência (Depends) pra ler o cabeçalho Authorization
2. Decodificar e validar a assinatura e a expiração do token JWT
3. Extrair o ID e o Perfil (Role) do token
4. Retornar 401 se o token for inválido ou ausente

---

## O que usamos (os componentes)

| Componente | De onde vem | Pra que serviu |
|---|---|---|
| `HTTPBearer` | fastapi.security | ler o cabeçalho `Authorization: Bearer <token>` |
| `HTTPAuthorizationCredentials` | fastapi.security | o objeto que carrega o token lido |
| `Depends` | fastapi | injetar a dependency na rota (rodar o porteiro antes) |
| `jwt.decode` | jose (python-jose) | validar a assinatura + expiração e ler o payload |
| `JWTError` | jose | o erro que o decode lança se o token for inválido/vencido |
| `HTTPException(401)` | fastapi | devolver o erro de não autorizado |

---

## Como funciona (o fluxo)

```
Cliente chama uma rota protegida com  Authorization: Bearer <token>
        ↓
FastAPI roda o get_current_user ANTES da rota (por causa do Depends)
        ↓
HTTPBearer le o cabecalho -> pega o token (ou None se nao tiver)
        ↓
  - sem token (None)        -> raise 401, a rota NEM roda
  - jwt.decode falha (JWTError: assinatura errada ou expirado) -> raise 401
  - tudo ok                 -> extrai id (sub) e role, retorna {id, role}
        ↓
A rota roda e recebe o usuario logado no parametro
```

O ponto-chave: se o porteiro lança 401, a rota **nunca executa**. É isso que "proteger" significa.

---

## Onde cada coisa está e POR QUÊ

### `get_current_user` -> em `back/core/security.py`
**Por que aqui:** o `security.py` é o lugar da **segurança/autenticação**. Ele já tinha o hash de senha e a **geração** do JWT (`create_access_token`). O `get_current_user` é a **validação** do JWT - é o par natural do create. Tudo que mexe com token/segredo fica num lugar só, e fica **reutilizável** (qualquer rota importa de lá).

### `security_scheme = HTTPBearer(auto_error=False)` -> também no `security.py`
**Por que aqui:** é o "leitor do cabeçalho", parte da mesma lógica de auth.
**Por que `auto_error=False`:** por padrão o HTTPBearer devolve **403** quando não tem token. A gente queria **401** (item 4 do card). Com `auto_error=False`, quando não tem token o `cred` vem `None`, e a gente trata na mão pra devolver 401.

### A rota `GET /me` -> em `back/routes/me_routes.py`
**Por que aqui:** rotas (endpoints) ficam em `routes/`. É uma rota de **demonstração** - serve pra provar que a proteção funciona (devolve quem está logado).

### `Depends(get_current_user)` -> dentro da rota
**Por que:** é o que "pendura" o porteiro na rota. Sem isso, a rota seria pública. Com isso, ela exige o token. Pra proteger qualquer outra rota, é só adicionar esse `Depends`.

### `app.include_router(me_routes.router)` -> no `main.py`
**Por que aqui:** o `main.py` é onde **todas** as rotas são registradas na aplicação. Criar a rota num arquivo não basta - tem que registrar no main.

---

## Os 4 itens do checklist -> onde estão

Uma única função (`get_current_user`) cobre os 4:

| # | Item | Linha/parte |
|---|---|---|
| 1 | Ler o Authorization | `HTTPBearer` + `Depends(security_scheme)` |
| 2 | Validar assinatura + expiração | `jwt.decode(...)` no try/except `JWTError` |
| 3 | Extrair ID e Role | `payload.get("sub")` e `payload.get("role")` |
| 4 | 401 se inválido/ausente | `raise erro_401` (no None e no JWTError) |

---

## Decisões técnicas

- **HTTPBearer (não OAuth2PasswordBearer):** nosso login usa JSON (email/senha), não o formato form do OAuth2. O HTTPBearer só lê o header `Bearer` e combina melhor - no Swagger, o botão "Authorize" mostra um campo pra colar o token direto.
- **auto_error=False:** pra devolver 401 (não 403) quando o token está ausente.
- **401 vs 403:** 401 = não autenticado (sem token / token ruim). 403 = autenticado, mas sem permissão (vai aparecer quando a gente fizer rotas só-de-admin).

---

## Como testar (reproduzir)

1. `uvicorn main:app --reload`  ->  `/docs`
2. `GET /me` sem token -> 401
3. `POST /login` (admin@ifba.edu.br / admin123) -> copia o access_token
4. Botão "Authorize" (cadeado) -> cola o token -> Authorize
5. `GET /me` de novo -> 200 com { id, role }
6. Token adulterado -> 401

---

## Aprendizados

### O JWT_SECRET tem que estar no .env (não o fallback)
A linha `os.getenv("JWT_SECRET", "dev-secret-...")` lê o segredo do .env, com um fallback. O fallback está no código (público no GitHub) - se fosse usado em produção, qualquer um poderia **forjar** um token (ex: role=admin). Por isso geramos um JWT_SECRET aleatório e colocamos no .env. (a assinatura do JWT só é segura se o segredo for secreto)

### Trocar o segredo invalida os tokens antigos
Quando mudamos o JWT_SECRET, os tokens gerados com o segredo velho deixam de valer (a assinatura não bate mais). É só logar de novo. As senhas/hashes não são afetadas (hash usa bcrypt, não o JWT_SECRET).

### O servidor lê o .env no startup
Mudou o .env? Reinicia o uvicorn - o `--reload` reinicia em mudança de `.py`, mas não em mudança de `.env`.

---

## Como proteger qualquer rota agora (o reaproveitamento)

Pra proteger uma rota existente, é só adicionar o `Depends(get_current_user)`:
```python
@router.get("/")
def listar_alunos(db: Session = Depends(get_db), usuario: dict = Depends(get_current_user)):
    ...
```
Sem token válido, a rota nem roda. E como a gente tem a `role` no `usuario`, dá pra fazer rotas só-de-admin (próximo passo da segurança).

---

## Links

- [[Proteger Rotas com JWT (guia)]] - o guia/passo a passo
- [[RF001 - Registro de implementação (o que fizemos)]] - o login que gera o token
- [[Conceitos Python do Código (def, return, etc)]] - o que é Depends e os termos
- [[Estrutura de Pastas do Backend]] - por que cada arquivo fica onde fica
- Card no Trello: Proteger Rotas Privadas (Middleware JWT)
