---
title: Conceitos Python do Código
tags: [python, conceitos, estudo, backend, grupo-5]
---

# Conceitos Python do Código

Dicionário de **cada pedaço de Python** que aparece no nosso backend, com exemplos reais do código. Serve pra você entender e **explicar pra qualquer um** (grupo, professor).

> Lê de cima pra baixo na primeira vez. Depois usa como consulta ("o que era mesmo o `**`?").

---

## 1. A estrutura de uma função

### `def` — define uma função
Uma **função** é um bloco de código com **nome**, que você chama quando quiser (em vez de repetir o código).

```python
def hash_password(senha_pura):
    ...
```
- `def` = "estou **def**inindo uma função"
- `hash_password` = o **nome** dela
- `(senha_pura)` = o que ela **recebe** pra trabalhar (o parâmetro)

**Pra que serve:** dar um nome a uma tarefa e reaproveitar. Em vez de escrever o hash toda hora, você chama `hash_password("admin123")`.

### `return` — devolve um valor
`return` **manda um valor de volta** pra quem chamou a função. Depois do `return`, a função acaba ali.

```python
def hash_password(senha_pura):
    return pwd_context.hash(senha_pura)
```
Essa função **devolve** (return) o hash. Quem chamou recebe esse valor.

> Sem `return`, a função faz o trabalho mas não te dá nada de volta (devolve `None`).

### Parâmetro vs argumento
- **Parâmetro** = o nome na definição (`senha_pura`)
- **Argumento** = o valor real que você passa (`"admin123"`)

---

## 2. Type hints — as "dicas de tipo"

```python
def create_access_token(user_id: int, role: str) -> str:
```
- `user_id: int` → o user_id **deveria** ser um número inteiro
- `role: str` → o role **deveria** ser texto
- `-> str` → a função **devolve** texto

**Pra que serve:** são só **dicas** (o Python não obriga), mas documentam o código e fazem o VS Code te ajudar (autocompletar, avisar erro). Profissional usa sempre.

---

## 3. Os tipos básicos

| Tipo | O que é | Exemplo no código |
|---|---|---|
| `str` | texto (string) | `"admin@ifba.edu.br"`, `"Ativo"` |
| `int` | número inteiro | `60`, `2024`, o `id` |
| `float` | número com vírgula | `nota_final` (7.5) |
| `bool` | verdadeiro/falso | `ativo = True`, o retorno do `verify_password` |
| `dict` | dicionário (chave→valor) | `payload = {"sub": ...}` |
| `list` | lista de coisas | `list[AlunoResponse]` |

---

## 4. `class` — define um "molde"

Uma **classe** é um molde que descreve um tipo de coisa. Os **models** e os **schemas** são classes.

```python
class Usuario(Base):
    __tablename__ = "usuarios"
    id = Column(Integer, primary_key=True)
```
- `class Usuario(Base)` = "defino uma coisa chamada **Usuario**, baseada em **Base**" (o Base do SQLAlchemy)
- Tudo indentado dentro = as características dela (as colunas)

**Pra que serve:** o `Usuario` vira o "desenho" da tabela. Cada linha do banco é um objeto desse molde.

---

## 5. `import` — trazer código de fora

```python
from jose import jwt
from passlib.context import CryptContext
import models.user_model
```
- `from jose import jwt` → da biblioteca `jose`, traz só a ferramenta `jwt`
- `import models.user_model` → traz o arquivo inteiro `user_model`

**Pra que serve:** reaproveitar código que outras pessoas (ou você) já escreveram, sem reescrever.

---

## 6. Datas e tempo (no `create_access_token`)

```python
expira = datetime.now(timezone.utc) + timedelta(minutes=60)
```
- `datetime.now(timezone.utc)` → **que horas são agora** (no fuso UTC, o padrão mundial)
- `timedelta(minutes=60)` → **uma duração** de 60 minutos
- somando os dois → "agora **+** 60 min" = **quando o token expira**

**Pra que serve:** definir a validade do token. Passou de `expira`, o token morre.

---

## 7. Dicionários e o `**`

### `{ }` — dicionário (chave: valor)
```python
payload = {"sub": str(user_id), "role": role, "exp": expira}
```
Guarda pares **chave → valor**. Aqui: `sub`→o id, `role`→o perfil, `exp`→a validade. É o que vai **dentro** do token.

### `**` — "desempacota" um dicionário
```python
aluno = Aluno(**dados.model_dump())
```
O `**` **abre** o dicionário e passa cada par como argumento. Se `dados` tem `{nome: "Maria", email: "x"}`, vira `Aluno(nome="Maria", email="x")`.

**Pra que serve:** evita escrever campo por campo na mão.

---

## 8. Decorators — o `@`

```python
@router.post("/login")
def login(...):
    ...
```
O `@` é um **decorator** — ele **gruda um comportamento** na função logo abaixo. Aqui ele diz: "essa função responde a um **POST** no endereço **/login**".

**Pra que serve:** sem o `@router.post`, a função `login` seria só uma função comum que ninguém chama pela internet. Com ele, vira um **endpoint**.

---

## 9. Coisas do FastAPI

| Peça | O que faz |
|---|---|
| `APIRouter()` | cria um "agrupador" de rotas (você junta as rotas de aluno num router, as de auth em outro) |
| `Depends(get_db)` | **injeta** a conexão do banco na rota automaticamente |
| `HTTPException(...)` | devolve um **erro HTTP** controlado (ex: 401, 409) |
| `response_model=...` | diz **o formato** da resposta (o FastAPI converte e valida) |
| `status_code=201` | define o código de sucesso (201 = criado) |

---

## 10. Coisas do SQLAlchemy (o banco)

### `Column(...)` — define uma coluna da tabela
```python
email = Column(String(255), unique=True, nullable=False)
```
- `String(255)` → texto de até 255 caracteres
- `unique=True` → não pode repetir
- `nullable=False` → não pode ficar vazio

### `relationship(...)` — liga duas tabelas
```python
matriculas = relationship("Matricula", back_populates="aluno")
```
Diz que um aluno tem várias matrículas (a "ponte" entre as tabelas).

### A busca: `query` → `filter` → `first`
```python
db.query(Usuario).filter(Usuario.email == dados.email).first()
```
- `.query(Usuario)` → "quero buscar **usuários**"
- `.filter(Usuario.email == x)` → "**onde** o email seja x"
- `.first()` → "me dá **o primeiro** (ou `None` se não achar)"

### Salvar: `add` → `commit` → `refresh`
```python
db.add(aluno)      # coloca na "fila" pra salvar
db.commit()        # confirma e grava no banco
db.refresh(aluno)  # recarrega (pega o id que o banco gerou)
```

---

## 11. Senha e token (passlib + jose)

### `CryptContext` / `pwd_context`
```python
pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")
```
`pwd_context` é o **motor de hash**. Ele sabe gerar (`.hash()`) e conferir (`.verify()`) usando o algoritmo **bcrypt**.

### `.encode()` (o `jwt.encode`)
```python
jwt.encode(payload, JWT_SECRET, algorithm="HS256")
```
`.encode()` = **"codificar/empacotar"**. Ele pega o payload, **assina** com o segredo, e devolve a string `eyJ...` (o token).

---

## 12. Lógica e erros

### `if`, `not`, `or`
```python
if not usuario or not verify_password(dados.senha, usuario.senha_hash):
```
- `not usuario` → "se **não** achou usuário"
- `or` → "**ou**"
- juntos → "se não achou usuário **OU** a senha não bate" → cai no erro

### `raise` — lança um erro
```python
raise HTTPException(status_code=401, detail="Credenciais inválidas")
```
`raise` **"joga" um erro** que interrompe a função na hora. O FastAPI captura e devolve o 401 pro cliente.

---

## 13. `yield` (no `get_db`)

```python
def get_db():
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()
```
- `yield` é um **`return` especial**: ele **entrega** o `db` pra rota usar, mas **pausa** aqui
- quando a rota termina, o código volta e cai no `finally`, que **fecha** a conexão

**Pra que serve:** garante que toda conexão com o banco é **fechada** depois do uso, sempre — mesmo se der erro no meio.

---

## Juntando tudo: a função de login linha a linha

```python
@router.post("/login", response_model=TokenResponse)   # decorator: POST /login, resposta = TokenResponse
def login(dados: LoginRequest, db: Session = Depends(get_db)):   # recebe os dados + a conexão do banco
    usuario = db.query(Usuario).filter(Usuario.email == dados.email).first()   # busca o usuário pelo email

    if not usuario or not verify_password(dados.senha, usuario.senha_hash):   # se não achou OU senha errada
        raise HTTPException(status_code=401, detail="Credenciais inválidas")   # lança erro 401

    token = create_access_token(user_id=usuario.id, role=usuario.role)   # gera o token JWT
    return TokenResponse(access_token=token)   # devolve o token
```

Se você consegue explicar **essas 6 linhas** usando os conceitos acima, você entende o código de verdade.

---

## Links

- [[Entendendo FastAPI e CORS]] — o que é FastAPI/CORS
- [[RF001 - Login com JWT]] — o login completo
- [[Modelagem do Banco de Dados]] — os models
- Documentação Python (PT-BR): https://docs.python.org/pt-br/3/
