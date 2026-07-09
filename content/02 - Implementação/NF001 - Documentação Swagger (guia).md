---
title: NF001 - Documentação Swagger/OpenAPI (guia)
tags: [nf001, swagger, openapi, documentacao, sprint-3, guia]
status: em andamento
sprint: 3
card_trello: "[NF001] Documentação Swagger/OpenAPI"
---

# NF001 - Documentação Swagger/OpenAPI (guia)

> Revisão final da documentação automática do FastAPI, pra entregar o projeto com cara profissional.

## O que o card pede

1. ✅ Acessar a rota `/docs` (já vem de graça no FastAPI).
2. Todas as rotas com **títulos e descrições** claras.
3. **Exemplos** práticos nos schemas do Pydantic.
4. O **cadeado do JWT** (Security Scheme) ativo no Swagger.

## A boa notícia: o FastAPI já dá 80% de graça

O `/docs` (Swagger) e o `/redoc` são gerados **automaticamente** a partir do código. A NF001 é **polir** o que já existe — não é construir do zero.

## Item 2 — títulos e descrições nas rotas

Em cada rota, dá pra adicionar `summary` (título curto) e uma descrição (a **docstring** da função vira a descrição):
```python
@router.post(
    "/",
    response_model=DisciplinaResponse,
    status_code=201,
    summary="Cadastrar disciplina",
)
def criar_disciplina(...):
    """Cria uma disciplina no sistema. **Só admin.** Retorna 409 se o código já existir."""
    ...
```
- `summary=` → o **título** curto que aparece na lista do Swagger.
- A **docstring** (entre `"""`) → a **descrição** expandida. Aceita markdown.

## Item 3 — exemplos nos schemas

No Pydantic v2, um exemplo por schema (aparece **preenchido** no "Try it out"):
```python
class DisciplinaCreate(BaseModel):
    model_config = ConfigDict(
        json_schema_extra={
            "example": {
                "nome": "Banco de Dados",
                "codigo": "BD101",
                "carga_horaria": 60,
                "ementa": "Modelagem e SQL",
            }
        }
    )
    nome: str = Field(min_length=2)
    codigo: str = Field(min_length=2)
    carga_horaria: int = Field(gt=0)
    ementa: str | None = None
```
Aí quem for testar no /docs já vê um JSON de exemplo pronto (não precisa adivinhar o formato).

## Item 4 — o cadeado do JWT

**Provavelmente já está funcionando.** A gente usa `HTTPBearer` no `get_current_user`, e o FastAPI já coloca o botão **Authorize** (o cadeado) no Swagger por causa disso. É só confirmar no `/docs`: tem que ter o botão "Authorize" no canto superior, onde você cola o token. Se estiver lá (e a gente já usou nos testes), **o item 4 está pronto**.

## Onde aplicar

- **Rotas** (item 2 — `summary` + docstring): auth, aluno, docente, disciplina, turma, matrícula, relatório, log, me.
- **Schemas** (item 3 — `example`): aluno, docente, disciplina, turma, matrícula, log, auth.

## O resultado

Um `/docs` onde cada rota tem **título + descrição**, cada schema tem **exemplo**, e o **cadeado de login** funciona — pronto pra apresentar pro professor.

## Links

- [[Entendendo FastAPI e CORS]] — o que é o FastAPI (e o /docs automático)
- [[Estrutura de Pastas do Backend]] — onde ficam rotas e schemas
- Card: [NF001] Documentação Swagger/OpenAPI
