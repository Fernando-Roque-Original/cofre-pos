---
title: RF003 - Registro de implementação
tags: [rf003, docente, professor, cadastro, registro, entrega, grupo-5]
status: concluído
sprint: 1
---

# RF003 - Como implementamos o Cadastro de Docente (registro)

> Status: concluído (os 3 itens do checklist do Trello).
> Decisão: Opção A - cadastrar um docente cria o login (Usuario, role docente) junto.

Documento de tudo que foi feito, as tecnologias/métodos usados, e os aprendizados (que foram muitos nesse).

---

## O que o RF003 pedia

1. Schema do Pydantic validando a entrada (Nome, CPF, Titulação, Email, Senha)
2. Rota POST /docentes com hash de senha
3. Tratar exceção para dados inválidos ou duplicados

---

## O fluxo construído

```
1. Cliente manda:  POST /docentes  { nome, cpf, titulacao, email, senha }
        ↓
2. Pydantic valida (DocenteCreate) - tamanho mínimo dos campos
        ↓
3. Checa CPF ou email duplicado  →  se sim, 409
        ↓
4. Hash da senha (bcrypt)
        ↓
5. Cria o Usuario (login, role docente) E o Docente, ligados pela FK
        ↓
6. Devolve o docente (DocenteResponse, sem senha) com status 201
```

---

## Tecnologias e métodos usados

| Tecnologia / método | Pra que serviu no RF003 |
|---|---|
| SQLAlchemy (model) | definir as colunas novas (cpf, titulacao) na tabela docentes |
| Alembic (migration) | aplicar o ALTER TABLE no banco sem perder dados |
| Pydantic (schema) | validar a entrada e formatar a saída (DocenteCreate / DocenteResponse) |
| passlib + bcrypt | hash da senha do docente |
| FastAPI (router) | criar a rota POST /docentes |
| Transação (db.flush + commit) | criar Usuario + Docente atomicamente (ou os dois, ou nenhum) |
| Git (branch + rebase) | isolar o trabalho numa branch baseada na do RF002 |

---

## O que foi feito (as 5 etapas)

### Etapa 1 - Model (docente_model.py)
Adicionadas duas colunas:
```python
cpf = Column(String(14), unique=True, nullable=False)
titulacao = Column(String(100), nullable=True)
```
**Por que:** o card pede CPF e Titulação. O `unique` no cpf serve pra checagem de duplicado.

### Etapa 2 - Migration (Alembic)
`alembic revision --autogenerate` gerou o `ALTER TABLE docentes ADD COLUMN cpf, titulacao`, e `alembic upgrade head` aplicou.
**Por que:** alterar a tabela que já existe sem destruir nada.

### Etapa 3 - Schema (docente_schema.py)
`DocenteCreate` (nome, cpf, titulacao, email, senha) com `Field(min_length=...)`, e `DocenteResponse` sem a senha.
**Por que:** validar o que entra (e bloquear campo vazio); nunca devolver senha.

### Etapa 4 - Rota (docente_routes.py)
`POST /docentes` com a Opção A: checa duplicado (409) → cria Usuario (role docente, senha em hash) → flush → cria Docente ligado → commit.
**Por que:** cadastrar docente cria o login dele junto, na tabela certa.

### Etapa 5 - Ligar no main.py
`from routes import ..., docente_routes` + `app.include_router(docente_routes.router)`.
**Por que:** sem o include, a rota não existe pra aplicação.

---

## Os 3 itens do checklist -> onde estão

| # | Item | Onde |
|---|---|---|
| 1 | Schema (Nome, CPF, Titulação, Email, Senha) | DocenteCreate (com Field) |
| 2 | Rota POST com hash | docente_routes.py + hash_password |
| 3 | Tratar inválido/duplicado | 422 (Pydantic, automático) + 409 (duplicado) |

---

## Os aprendizados (esse RF teve vários perrengues úteis)

### 1. relationship NÃO é coluna
O `turmas = relationship("Turma", ...)` no model não cria coluna no banco - é só navegação no Python. A ligação real é a ForeignKey. Por isso a relationship não entra na migration. (detalhe em [[Modelagem do Banco de Dados]])

### 2. Migrations do Alembic formam uma FILA (chain)
A migration do RF003 vem **depois** da do RF002 (`down_revision = 808c99c1db83`). Como o banco já tinha o RF002 aplicado, a branch do RF003 precisou ser feita **a partir da branch do RF002**, não da main. Branch da main dava o erro `Can't locate revision`.
**Lição:** se a feature B mexe no banco depois da feature A, a branch de B nasce da branch de A (ou A merge primeiro).

### 3. O .env é local de cada um
O `.env` é gitignorado - não vai pro Git. Quando ele sumiu (numa troca de branch), foi só **recriar a partir do `.env.example`** com a senha local. Cada pessoa que clonar faz isso.
**Lição:** segredo (senha do banco) fica local; o `.env.example` é o molde pra recriar.

### 4. O schema tem que BATER com o model
A gente quase colocou `data_nascimento` no schema do docente - mas o model/tabela não tem essa coluna. Se ficasse, quebraria. O que está no schema precisa existir no model.
**Lição:** schema e model andam juntos. Adicionar campo = mexer nos dois (+ migration).

### 5. PR de fork mira o repo errado por padrão
Ao abrir um PR a partir do fork (hiag-code), o GitHub coloca o repo do professor (DjanInfo) como base. Tem que trocar o "base repository" pra **hiag-code/main** (o fork de vocês).
**Lição:** sempre conferir a base do PR quando o repo é um fork.

### 6. Pasta certa pra cada comando
- `git` roda na **raiz** do repo (`poswebreactdeploy`)
- `alembic` e `uvicorn` rodam em **`back/`**
Rodar no lugar errado dá "not a git repository" ou "path not found". (e cuidado pra não digitar comando dentro do `.env` - aconteceu um `cd back` parar lá dentro)

---

## Como testar (reproduzir)

1. `cd back && alembic upgrade head`
2. `uvicorn main:app --reload` -> `/docs`
3. POST /docentes (nome, cpf, titulacao, email, senha) -> 201 (sem senha)
4. POST /login com o email/senha do docente -> 200 (login nasce junto, role docente)
5. Repetir o cadastro -> 409
6. Senha vazia -> 422

Verificado no banco: o docente fica em `docentes` e o login em `usuarios` (role docente), ligados por `usuario_id`.

---

## Decisões tomadas (análise crítica - o card não é absoluto)

- **Rota `/docentes`** (não `/professores` como o card diz) - consistente com o model e o backend do professor. Alinhar com o grupo.
- **409 para duplicado** (não 422 como o card diz) - 409 é a convenção pra conflito; o 422 vem de graça do Pydantic pros inválidos.
- **Sem `data_nascimento`** - o card não pede; mantivemos o docente mínimo. Dá pra adicionar depois (model + migration + schema).

---

## Links

- [[RF003 - Cadastro de Docente (guia)]] - o guia/passo a passo
- [[RF002 - Registro de implementação (o que fizemos)]] - o gêmeo (aluno), mesmo padrão
- [[Modelagem do Banco de Dados]] - ForeignKey vs relationship
- [[Tecnologias e Bibliotecas do Projeto]] - o que cada tecnologia faz
- [[Estrutura de Pastas do Backend]] - onde cada arquivo fica
