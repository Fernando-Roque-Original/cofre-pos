---
title: NF001 - Registro de implementação (o que fizemos)
tags: [nf001, swagger, openapi, documentacao, registro, sprint-3, backend]
status: concluído
sprint: 3
card_trello: "[NF001] Documentação Swagger/OpenAPI"
---

# NF001 - Registro de implementação (o que fizemos)

> Status: concluído (4/4). Verificado direto no `/openapi.json`.

Revisão final da documentação automática do FastAPI, pra entregar com cara profissional.

## O que o card pedia

1. ✅ Acessar `/docs` (automático).
2. Títulos e descrições claras nas rotas.
3. Exemplos nos schemas do Pydantic.
4. Cadeado do JWT ativo no Swagger.

## O que fizemos

| Item | O que | Onde |
|---|---|---|
| 2 | `summary` (título) + docstring (descrição) em **todas** as rotas | os 9 arquivos de `routes/` |
| 3 | `example` (`json_schema_extra`) nos schemas de **entrada** | aluno, docente, disciplina, turma, matrícula, auth |
| 4 | **já funcionava** — confirmamos o cadeado | `HTTPBearer` no `security.py` |
| bônus | título + **descrição** + versão da API (topo do /docs) | `main.py` (`FastAPI(...)`) |

## As técnicas (do guia)

- **`summary="..."`** no decorator → o título curto na lista do Swagger.
- **docstring** (`""" ... """`) na função → a descrição expandida (aceita markdown).
- **`model_config = ConfigDict(json_schema_extra={"example": {...}})`** no schema → um JSON de exemplo já preenchido no "Try it out".

## A verificação (no /openapi.json)

```
API: API Pós-Graduação IFBA v1.0.0
securitySchemes: HTTPBearer          ← o cadeado (item 4)
14 rotas com summary                 ← item 2
DisciplinaCreate.example: {nome, codigo, carga_horaria, ementa}   ← item 3
```

## Os 4 itens do checklist

| # | Item | Como ficou |
|---|---|---|
| 1 | Acessar /docs | automático do FastAPI |
| 2 | Títulos/descrições | `summary` + docstring em todas as rotas |
| 3 | Exemplos nos schemas | `example` nos schemas de entrada |
| 4 | Cadeado JWT | `HTTPBearer` (o botão Authorize já aparece) |

## Aprendizados

- O FastAPI **gera o /docs sozinho** — a NF001 é **polir**, não construir.
- `summary` = título; **docstring** = descrição. Simples e poderoso.
- `example` no schema economiza tempo de quem testa (não precisa adivinhar o JSON).
- `HTTPBearer` já entrega o cadeado de graça — item 4 vinha "de brinde" desde o "Proteger Rotas".

## Links

- [[NF001 - Documentação Swagger (guia)]] - o guia/roteiro
- [[Entendendo FastAPI e CORS]] - o /docs automático
- Card: [NF001] Documentação Swagger/OpenAPI
