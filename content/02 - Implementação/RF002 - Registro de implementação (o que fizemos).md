---
title: RF002 - Registro de implementação
tags: [rf002, aluno, cadastro, registro, entrega, grupo-5]
status: concluído
sprint: 1
---

# RF002 - Como implementamos o Cadastro de Aluno (registro)

> Status: concluído (os 4 itens do checklist do Trello).
> Decisão tomada: Opção A - cadastrar um aluno cria o login (Usuario) junto.

Documento do que foi criado, mudado e adicionado pra fazer o cadastro de aluno - e por quê.

---

## O que o RF002 pedia

1. Schema do Pydantic validando a entrada (Nome, CPF, Email, Data de Nasc, Senha)
2. Hash da senha (passlib)
3. Rota POST /alunos pra inserir no banco
4. Tratar exceção (409/400) se CPF ou Email já existir

---

## O fluxo que construímos

```
1. Cliente manda:  POST /alunos  { nome, cpf, email, data_nascimento, senha }
        ↓
2. Pydantic valida (AlunoCreate) - inclusive tamanho mínimo dos campos
        ↓
3. Checa se CPF ou email já existe  →  se sim, 409
        ↓
4. Hash da senha (bcrypt)
        ↓
5. Cria o Usuario (login, role estudante) E o Aluno, ligados pela FK
        ↓
6. Devolve o aluno (AlunoResponse, sem a senha) com status 201
```

---

## Arquivos - o que mudamos

| Arquivo | Tipo | O que mudou |
|---|---|---|
| back/models/aluno_model.py | modificado | adicionadas colunas cpf e data_nascimento; matricula virou opcional |
| back/alembic/versions/...adiciona_cpf...py | novo | migration que altera a tabela alunos |
| back/schemas/aluno_schema.py | modificado | AlunoCreate e AlunoResponse com os campos novos + validação |
| back/routes/aluno_routes.py | modificado | rota POST /alunos com a Opção A (cria login + aluno) |

---

## Cada mudança, explicada (o que e por que)

### Model Aluno
Adicionamos:
- `cpf` (String(14), unique, obrigatório)
- `data_nascimento` (Date, opcional)
- `matricula` virou `nullable=True` (opcional)

**Por que:** o card pede CPF e Data de Nascimento. A matrícula não é pedida no cadastro, então deixou de ser obrigatória.

### Migration Alembic
Gerada com `alembic revision --autogenerate`. Ela faz `ALTER TABLE alunos ADD COLUMN cpf...` e `ADD COLUMN data_nascimento...`.

**Por que:** alterar uma tabela que já existe sem perder dados. O `create_tables.py` não conseguia fazer isso (só criava tabela nova).

### Schema (aluno_schema.py)
- `AlunoCreate`: nome, cpf, email, data_nascimento, senha - cada um com `Field(min_length=...)` pra rejeitar valores vazios
- `AlunoResponse`: id, nome, cpf, email, data_nascimento, status - **sem a senha**

**Por que:** validar o que entra (e bloquear campo vazio); e nunca devolver senha numa resposta de API.

### Rota (aluno_routes.py) - a Opção A
1. Checa CPF ou email duplicado (na tabela alunos e na usuarios) -> 409
2. Cria o `Usuario` com a senha em hash e `role="estudante"`
3. `db.flush()` pra obter o `usuario.id`
4. Cria o `Aluno` ligado (`usuario_id`)
5. `db.commit()` grava os dois juntos (transação)

**Por que:** cadastrar um aluno cria o login dele junto, com a senha na tabela certa (usuarios), não em alunos.

---

## Os 4 itens do checklist -> onde estão no código

| # | Item | Onde |
|---|---|---|
| 1 | Schema validando entrada | aluno_schema.py (AlunoCreate com Field) |
| 2 | Hash da senha | hash_password(dados.senha) na rota |
| 3 | Rota POST /alunos | aluno_routes.py |
| 4 | 409 se CPF/email duplicado | os dois if ... raise HTTPException(409) |

---

## Problema que encontramos (e resolvemos)

**Senha vazia passava.** Mandando `"senha": ""`, o cadastro era aceito (201).
- **Causa:** `senha: str` só verifica que é texto - e `""` é um texto válido (vazio).
- **Fix:** `senha: str = Field(min_length=4)` - agora vazio é rejeitado com 422 automático.
- **Lição:** validar tipo não é o mesmo que validar conteúdo. Foi um achado de análise crítica.

---

## Como testar (reproduzir)

1. `cd back && alembic upgrade head`
2. `uvicorn main:app --reload`  ->  `/docs`
3. POST /alunos com nome, cpf, email, data_nascimento, senha -> 201 (sem senha na resposta)
4. POST /login com o email/senha do aluno -> 200 (o login nasceu junto)
5. Repetir o cadastro -> 409
6. Mandar senha vazia -> 422

Verificado no banco: o aluno fica em `alunos` e o login em `usuarios` (role estudante, senha em hash), ligados por `usuario_id`.

---

## O que é RF002 e o que não é (escopo)

| | |
|---|---|
| É RF002 | o cadastro de aluno (que cria o login junto) |
| Não é RF002 | proteger rotas com o token (Sprint 2); validar formato de CPF/email (melhoria opcional) |

---

## Links

- [[RF002 - Cadastro de Aluno (guia)]] - o guia/passo a passo
- [[Estrutura de Pastas do Backend]] - onde cada arquivo fica
- [[Conceitos Python do Código (def, return, etc)]] - os termos do código
- [[RF001 - Registro de implementação (o que fizemos)]] - o registro do login
