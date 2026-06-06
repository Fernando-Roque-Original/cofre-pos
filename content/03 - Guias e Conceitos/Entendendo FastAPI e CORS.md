---
title: Entendendo FastAPI e CORS
tags: [conceitos, fastapi, cors, backend, grupo-5]
---

# Entendendo FastAPI e CORS

Nota de conceito pra quem quiser entender o que é o FastAPI e o que aquele bloco de CORS faz no `main.py`. Sem isso dá pra programar copiando, mas entendendo você não fica perdido.

---

## Parte 1 — O que é FastAPI

FastAPI é um **framework Python pra construir APIs**.

- **API** = a parte do sistema que recebe pedidos e devolve respostas, **sem tela**. É o "cérebro" no servidor. O React (a tela) pede, a API responde.
- **Framework** = ferramentas prontas; você não escreve tudo do zero.

### O que ele faz na prática

1. **Recebe requisições** do React (ex: "lista os alunos", "faz login")
2. **Roda a lógica Python** (valida, decide)
3. **Conversa com o banco** (via SQLAlchemy)
4. **Devolve resposta** em **JSON**
5. De brinde: **documentação automática** (`/docs`) e **validação** (Pydantic)

### Onde ele entra no nosso projeto

```
[ React (src/) ]                ← a tela
      │  requisição HTTP
      ▼
[ FastAPI (back/) ]             ← recebe, valida, decide
      │  SQLAlchemy
      ▼
[ PostgreSQL (agents_db) ]      ← o banco que modelamos
```

Tudo que modelamos (models + 6 tabelas) é a **fundação**. O FastAPI é quem **expõe** essa fundação pro mundo, criando os **endpoints** que o React usa.

> O backend do professor em Node (`poswebserver`) faz exatamente isso — recebe em `/alunos`, `/docentes` e devolve JSON. O FastAPI é a **versão Python** disso.

### Analogia do restaurante

- **Cliente** = React (faz o pedido)
- **Garçom** = FastAPI (anota, leva pra cozinha, traz o prato)
- **Cozinha** = banco de dados
- **Conferir o pedido** = validação (Pydantic)

O FastAPI é o garçom: faz a ponte entre a tela e o banco.

### As peças e onde ficam no `back/`

| Peça | O que é | Arquivo |
|---|---|---|
| App | a aplicação | `back/main.py` (`app = FastAPI()`) |
| Rotas / endpoints | as "portas" (URLs) | `back/routes/` |
| Schemas (Pydantic) | validam entrada/saída | `back/schemas/` |
| Models (SQLAlchemy) | as tabelas | `back/models/` |
| Swagger | doc automática | aparece em `/docs` |

### Ciclo de toda requisição

```
recebe  →  valida  →  mexe no banco  →  responde
```

Todo endpoint (login, alunos, docentes...) segue esse mesmo ciclo.

---

## Parte 2 — O que é CORS e como foi configurado

### O problema que o CORS resolve

Uma **origem** = `protocolo + domínio + porta`. Exemplo:

```
http://localhost:5173   ← o React (front)
http://localhost:8000   ← o FastAPI (back)
```

Mesmo os dois sendo "localhost", a **porta é diferente** → o navegador trata como **origens diferentes**. Por padrão, o navegador **bloqueia** uma origem de chamar a outra (segurança). O CORS é o que libera.

### O que foi adicionado no `main.py`

Duas coisas — o import e o bloco do middleware:

```python
from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware   # ① import

app = FastAPI(title="API Pós-Graduação IFBA")

app.add_middleware(                                   # ② middleware
    CORSMiddleware,
    allow_origins=[
        "https://djansantos.com.br",   # produção
        "http://localhost:5173",       # Vite em dev
    ],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

@app.get("/")
def home():
    return "minha api"
```

### O que cada parâmetro faz

| Parâmetro | Função |
|---|---|
| `allow_origins` | a **lista de sites** liberados a chamar a API |
| `allow_credentials` | permite enviar cookies / token de auth |
| `allow_methods=["*"]` | libera todos os métodos (GET, POST, PUT, DELETE...) |
| `allow_headers=["*"]` | libera todos os cabeçalhos (ex: o `Authorization` do JWT) |

`["*"]` = curinga, significa "todos".

### O que é "middleware"

Uma **camada que passa em toda requisição**, antes de chegar na rota e depois de sair. Tipo um porteiro. O `CORSMiddleware` adiciona **cabeçalhos de permissão** nas respostas.

### Como funciona no navegador

```
1. React (5173) tenta:  fetch("http://localhost:8000/alunos")
        ↓
2. Navegador pergunta antes (preflight):
   OPTIONS /alunos  →  "o 5173 pode te chamar?"
        ↓
3. CORSMiddleware responde:
   Access-Control-Allow-Origin: http://localhost:5173
        ↓
4. Navegador libera a requisição real
        ↓
   (se não estivesse na lista → navegador BLOQUEIA)
```

> [!important] Quem bloqueia é o navegador, não a API
> O CORS não "abre uma porta" — ele só adiciona os headers de permissão, e o **navegador** decide. Por isso, testar pelo **Swagger ou Postman funciona sem CORS** (eles não são navegadores). O CORS só importa quando é o **React no navegador** chamando a API.

### No nosso projeto

- **Dev:** React em `localhost:5173` (Vite) → na lista
- **Produção:** `djansantos.com.br` → na lista
- Se o front mudar de endereço, é só adicionar em `allow_origins`.

---

## Resumo em uma frase

> Modelamos **onde os dados moram** (banco + models). O **FastAPI deixa esses dados acessíveis** (endpoints), e o **CORS libera o React** a conversar com esses endpoints pelo navegador.

---

## Links

- FastAPI docs: https://fastapi.tiangolo.com/
- CORS no FastAPI: https://fastapi.tiangolo.com/tutorial/cors/
- Notas: [[Modelagem do Banco de Dados]] · [[RF001 - Login com JWT]] · [[Como começar - Sprint 1]]
