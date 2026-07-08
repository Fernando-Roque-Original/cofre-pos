---
title: RF005 - Registro de implementação (o que fizemos)
tags: [rf005, turma, fastapi, jwt, autorizacao, registro, sprint-2, backend]
status: concluído
sprint: 2
card_trello: "[RF005] Cadastro de Turma (POST /turmas)"
---

# RF005 - Registro de implementação (o que fizemos)

> Status: concluído (4/4) — feito em dupla (implementação + revisão antes do merge).
> Rota `POST /turmas`, protegida (admin), validando que a disciplina e o docente existem.

Registro do que foi feito no RF005. Esse foi **colaborativo**: uma primeira versão foi implementada, a gente revisou, e foi ajustada antes de mergear — fica registrado o que mudou e por quê.

## O que o card pedia

1. Criar schema Pydantic (ID da Disciplina, ID do Professor, Semestre, Vagas)
2. Proteger a rota com JWT
3. Verificar se os IDs do Professor e da Disciplina existem (404 se não)
4. Salvar a turma e retornar HTTP 201

## O que construímos (e por que cada coisa fica onde)

| Arquivo | O que tem | Por que aqui |
|---|---|---|
| `schemas/turma_schema.py` | `TurmaCreate` / `TurmaResponse` | contrato de entrada/saída; validações (`ge=1`, semestre `min/max`) |
| `routes/turma_routes.py` | `POST /turmas` | o endpoint; valida FK + salva |
| `models/turma_model.py` | + coluna `vagas_disponiveis` | preparado pra controlar vagas (usado no RF006) |
| `alembic/versions/...vagas_disponiveis` | migration | versiona a coluna nova; encadeada **depois** do RF004 |
| `main.py` | registra o router | sem isso, 404 |

## Como funciona (o fluxo)

```
Cliente: POST /turmas + token de admin
   ↓ Depends(require_admin)  → token vale? (401) → é admin? (403)
   ↓ FastAPI valida o corpo com TurmaCreate (senão 422)
   ↓ a disciplina (disciplina_id) existe?  → senão 404
   ↓ o docente (docente_id) existe?        → senão 404
   ↓ cria a Turma (vagas_disponiveis = vagas_total) → commit
   ↓ devolve 201 + TurmaResponse
```

## Conceito novo: validar chave estrangeira (FK) antes de salvar

A turma aponta pra uma disciplina e um docente por **id**. Antes de salvar, a gente confere se esses ids **existem** — senão devolve **404** com mensagem clara (sem isso, o banco estouraria um erro feio de FK).

## Decisões

> [!check] Proteção: `require_admin`
> Criar turma é tarefa da coordenação. O time combinou deixar **só admin** (igual no RF004), mesmo o card só falando "JWT".

> [!check] `vagas_disponiveis` (adiantou o RF006)
> Foi adicionada a coluna `vagas_disponiveis` (= `vagas_total` na criação). É a **opção A** que tínhamos mapeado pro RF006 — então o RF006 já começa com essa decisão tomada e a coluna pronta.

## O processo de revisão (o que a 1ª versão tinha e foi corrigido)

A primeira versão **funcionava**, mas a revisão (antes de mergear) pegou 3 coisas:

| Problema na 1ª versão | Correção |
|---|---|
| Um `409` "disciplina já possui turma" | **removido** — uma disciplina tem **várias** turmas (e isso travaria o RF006) |
| Proteção com `get_current_user` | trocado por `require_admin` (consistência com o RF004) |
| Branch desatualizada com a main | **rebaseada** — trouxe o `require_admin`, evitou conflito no `main.py` e encadeou a migration certo |

> [!tip] Lição
> **Revisar antes de mergear pega esse tipo de coisa.** O código rodava, mas tinha um erro de regra de negócio (o 409) e uma inconsistência de permissão. Code review funciona — e dá pra fazer no chão de fábrica, entre colegas.

## Os 4 itens do checklist -> onde ficam

| # | Item | Onde |
|---|---|---|
| 1 | Schema (ids, semestre, vagas) | `turma_schema.py` |
| 2 | Proteção JWT (admin) | `require_admin` na rota |
| 3 | IDs existem -> 404 | checagens de Disciplina/Docente |
| 4 | Salvar -> 201 | `add/commit/refresh` + `status_code=201` |

## Aprendizados

- **Validar FK antes de salvar** (404 amigável em vez de erro 500 do banco).
- **Consistência de autorização no time:** turma = admin, igual disciplina — vale alinhar quem pode o quê.
- **Migration encadeia na ordem:** a de `vagas_disponiveis` veio com `down_revision` apontando pro RF004 (`fe4caf...`), sem criar "duas cabeças".
- **Adiantar dependência:** o `vagas_disponiveis` já destrava o RF006 — pensar no próximo passo na hora ajuda.

## Links

- [[RF005 - Cadastro de Turma (guia)]] - o guia/roteiro
- [[RF006 - Efetuar Matrícula (guia)]] - o próximo, que usa as turmas e as vagas
- [[RF004 - Registro de implementação (o que fizemos)]] - o `require_admin` reusado aqui
- [[Modelagem do Banco de Dados]] - tabelas turmas/matriculas
- Card: [RF005] Cadastro de Turma (POST /turmas)
