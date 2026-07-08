---
title: RF006 - Registro de implementação (o que fizemos)
tags: [rf006, matricula, fastapi, jwt, transacao, registro, sprint-2, backend]
status: concluído
sprint: 2
card_trello: "[RF006] Efetuar Matrícula (POST /matriculas)"
---

# RF006 - Registro de implementação (o que fizemos)

> Status: concluído (6/6).
> Testado: 201 (matrícula) · 409 (duplicada) · 400 (sem vaga) · 403 (não-aluno) · 404 (turma inexistente).

## O que o card pedia

1. Rota protegida `POST /matriculas` que recebe o ID da Turma.
2. Pegar o ID do Aluno **de dentro do token** (não confiar no corpo).
3. Regra 1: não duplicar matrícula (409).
4. Regra 2: checar `vagas_disponiveis > 0`.
5. Salvar + diminuir 1 vaga (transação).
6. HTTP 400 se não houver vagas.

## O que construímos

| Arquivo | O que | Por que |
|---|---|---|
| `schemas/matricula_schema.py` | `MatriculaCreate` (**só** `turma_id`) + `MatriculaResponse` | o aluno NÃO vem do cliente (segurança) |
| `routes/matricula_routes.py` | `POST /matriculas` | toda a lógica (aluno do token, duplicado, vaga, transação) |
| `main.py` | registra o router | sem isso, 404 |

## Como funciona (o fluxo)

```
POST /matriculas + token  (turma_id no corpo)
   ↓ acha o Aluno pelo token: Aluno.usuario_id == logado["id"]  (senão 403)
   ↓ a turma existe?  (senão 404)
   ↓ ja matriculado nessa turma?  (se sim -> 409)
   ↓ vagas_disponiveis > 0?  (se nao -> 400)
   ↓ cria a matricula + turma.vagas_disponiveis -= 1  → UM commit (transacao)
   ↓ 201 + MatriculaResponse
```

## Os 2 pontos-chave

> [!check] 1. A identidade vem do TOKEN, não do corpo (segurança)
> O `MatriculaCreate` só tem `turma_id`. O aluno é descoberto a partir do usuário logado. Se confiasse no corpo, qualquer um matricularia outra pessoa.

> [!check] 2. O vínculo aluno↔usuário já era uma FK
> A gente ia ligar por email, mas descobriu que o `Aluno` já tem `usuario_id` (chave estrangeira pro Usuario). Então foi direto: `Aluno.usuario_id == logado["id"]`. Bem mais limpo.

## A transação

Salvar a matrícula **e** diminuir a vaga num **único `commit`** = ou os dois entram, ou nenhum. Nunca vai ter matrícula sem a vaga ter caído.

## Os 6 itens do checklist -> onde ficam

| # | Item | Onde |
|---|---|---|
| 1 | Rota protegida recebendo turma_id | `MatriculaCreate` + `Depends(get_current_user)` |
| 2 | Aluno do token | `Aluno.usuario_id == logado["id"]` |
| 3 | Não duplicar | busca aluno+turma -> 409 |
| 4 | Vagas > 0 | `if turma.vagas_disponiveis <= 0` |
| 5 | Salvar + -1 vaga | `add` + `-= 1` + um `commit` |
| 6 | 400 sem vaga | o `raise` da regra de vaga |

## Aprendizados

- **Identidade do token** — o padrão de segurança "não confie no que o cliente manda sobre quem ele é".
- **Transação** — mudanças relacionadas num `commit` só.
- **Detalhe de teste (trailing slash):** requisição pra `/rota` (sem barra) numa rota de coleção faz o FastAPI redirecionar (307), e o cliente **derruba o header Authorization** no redirect → 401 enganoso. Bater na URL com a barra (`/rota/`) resolve. Foi no teste, não no código.

## Links

- [[RF006 - Efetuar Matrícula (guia)]] - o guia/roteiro
- [[Proteger Rotas - Registro de implementação (o que fizemos)]] - o `get_current_user`
- [[RF005 - Registro de implementação (o que fizemos)]] - as turmas que a gente matricula
- Card: [RF006] Efetuar Matrícula (POST /matriculas)
