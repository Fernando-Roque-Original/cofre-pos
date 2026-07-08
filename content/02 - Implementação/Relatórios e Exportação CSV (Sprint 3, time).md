---
title: Relatórios e Exportação CSV (Sprint 3, time)
tags: [sprint-3, relatorios, csv, registro, backend, time]
status: concluído
sprint: 3
---

# Relatórios e Exportação CSV (Sprint 3, feito em time)

> Registro das features de relatório da Sprint 3 — feitas pelo time (Hiago e Ana) — com as revisões e correções que fizemos antes de mergear.

## Os relatórios (RF007, RF008, RF009) — feitos pelo Hiago

Todos são `GET`, protegidos por admin, no `routes/relatorio_routes.py`:

| Rota | O que mostra |
|---|---|
| `GET /relatorios/turmas` (RF007) | ocupação das turmas: vagas totais, matriculados, vagas disponíveis |
| `GET /relatorios/academico` (RF008) | ocupação **por disciplina** + taxa de ocupação geral (junta docente) |
| `GET /relatorios/matricula` (RF009) | lista as matrículas: nome do aluno, disciplina, código da turma, status |

Pro RF009, o Hiago também adicionou uma coluna **`codigo` na turma** (pra o relatório mostrar o código).

## Exportação CSV — feita pela Ana

- `GET /relatorios/turmas?formato=csv` → baixa um **arquivo CSV** em vez de JSON.
- Usa `StreamingResponse` + `Content-Disposition: attachment` pra **forçar o download** no navegador.
- Sem o `?formato=csv`, continua devolvendo JSON (comportamento padrão intacto).

## As correções que a gente fez (code review na prática)

> [!check] O que a revisão pegou antes do merge
> - **RF005 (turma):** tinha um `409` que travava "1 turma por disciplina" — errado (disciplina tem várias turmas). Removido.
> - **Migration do `codigo` na turma:** era `NOT NULL` sem default → quebrava o `upgrade` em bancos com turmas. Corrigimos: **nullable → backfill (TURMA-<id>) → NOT NULL**.
> - **CSV da Ana:** conflitava com a main (por causa de um dead code que a gente tinha limpado). Resolvemos com **cherry-pick, mantendo a autoria dela**.
> - **Limpezas cosméticas** nos relatórios (dead code, nomes, arredondamento).

## Aprendizados

- **Code review pega bug de verdade** — o 409 do RF005 e a migration `NOT NULL` teriam quebrado o time.
- **Migration numa tabela com dados** precisa de cuidado (nullable + backfill, nunca `NOT NULL` seco).
- **Conflito de PR de fork** dá pra resolver **preservando a autoria** de quem fez (cherry-pick + resolver).
- **Ordem de merge** importa quando há migration na jogada.

## Links

- [[NF006 - Registro de implementação (o que fizemos)]] - a auditoria (também Sprint 3)
- [[RF005 - Registro de implementação (o que fizemos)]] - o cadastro de turma que os relatórios usam
- [[Modelagem do Banco de Dados]] - as tabelas
