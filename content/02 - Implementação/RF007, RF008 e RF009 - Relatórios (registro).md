---
title: RF007, RF008 e RF009 - Relatórios (registro)
tags: [rf007, rf008, rf009, relatorios, csv, registro, sprint-3, backend]
status: concluído
sprint: 3
card_trello: "[RF007] Turmas/Ocupação · [RF008] Acadêmico · [RF009] Matrículas"
---

# RF007, RF008 e RF009 - Relatórios (registro)

> Os três relatórios da Sprint 3, feitos em time. Todos são `GET`, **restritos a admin** (`Depends(require_admin)`) e ficam no mesmo arquivo `routes/relatorio_routes.py`. Nenhum cria/edita dado — só **leem e resumem** o que já existe (turmas, matrículas, vagas).

## Visão geral

| Card | Rota | Pra que serve |
|---|---|---|
| RF007 | `GET /relatorios/turmas` | ocupação de cada turma (vagas x matriculados) — com opção de **CSV** |
| RF008 | `GET /relatorios/academico` | ocupação **agrupada por disciplina** + taxa geral do semestre |
| RF009 | `GET /relatorios/matricula` | lista todas as matrículas (aluno, disciplina, turma, status) |

Por que juntos: são a mesma família (leitura + `JOIN` + agregação com `func.count`), no mesmo arquivo, com a mesma proteção de admin.

## RF007 — Relatório de Turmas e Ocupação

**Rota:** `GET /relatorios/turmas` · **Acesso:** admin

Faz `JOIN` de turma com disciplina e um `outerjoin` com matrícula, agrupando por turma pra **contar os matriculados**. Devolve, por turma:

- `turma_id`, `disciplina`, `semestre`
- `total_de_vagas`, `matriculados`, `vagas_disponiveis`

Mais um `total_turmas` no topo. É o retrato de "quão cheia está cada turma".

### Exportação em CSV
- `GET /relatorios/turmas?formato=csv` → em vez de JSON, baixa um **arquivo CSV** (planilha).
- Usa `StreamingResponse` + o cabeçalho `Content-Disposition: attachment` pra **forçar o download** no navegador.
- Sem o `?formato=csv`, o comportamento padrão (JSON) fica intacto — a mesma rota serve os dois formatos.

## RF008 — Relatório Acadêmico

**Rota:** `GET /relatorios/academico` · **Acesso:** admin

Vai além do RF007: junta também o **docente** e **agrupa as turmas por disciplina**, calculando percentuais. Devolve:

- **Totais gerais:** `total_turmas`, `vagas_totais_ofertadas`, `total_matriculados`, `total_vagas_ociosas` e a **`taxa_de_ocupacao_geral`** (matriculados ÷ vagas × 100).
- **`detalhes_por_disciplina`:** cada disciplina com sua lista de turmas, e cada turma com `docente`, `semestre`, `vagas_total`, `vagas_disponiveis`, `matriculados` e o **`percentual_ocupacao`** daquela turma.

É a visão de gestão: onde está sobrando ou faltando vaga.

## RF009 — Relatório de Matrículas

**Rota:** `GET /relatorios/matricula` · **Acesso:** admin

`JOIN` de matrícula com aluno, turma e disciplina pra listar **cada matrícula**:

- `matricula_id`, `nome_aluno`, `id_aluno`, `data_de_matricula`
- `disciplina`, `codigo_disciplina`, `codigo_turma`, `status_matricula`

Mais um `total_matriculado`. É a lista "quem está matriculado em quê".

> Pra esse relatório mostrar o **código da turma**, foi preciso adicionar a coluna `codigo` na tabela `turmas` — o que gerou a correção de migração descrita abaixo.

## As correções que a revisão pegou (code review na prática)

> [!check] O que pegamos antes de mergear
> - **RF005 (turma):** tinha um `409` travando "1 turma por disciplina" — errado, disciplina tem **várias** turmas. Removido.
> - **Migração do `codigo` na turma:** era `NOT NULL` sem default → quebraria o `upgrade` em bancos que já tinham turmas. Corrigimos com o padrão seguro: **nullable → backfill (`TURMA-<id>`) → NOT NULL**.
> - **PR do CSV:** conflitava com a `main` por causa de um _dead code_ que já tínhamos limpado. Resolvido com **cherry-pick, preservando a autoria original** de quem fez.
> - **Limpezas cosméticas** nos relatórios (dead code, nomes, arredondamento).

## Aprendizados

- **Relatório = leitura + `JOIN` + agregação.** Nenhum deles escreve no banco; só resumem. Por isso são `GET` e cabem bem juntos.
- **A mesma rota pode servir JSON e CSV** — basta um parâmetro (`?formato=csv`) e um `StreamingResponse`.
- **Code review pega bug de verdade:** o `409` do RF005 e a migração `NOT NULL` teriam quebrado o time.
- **Migração em tabela com dados** precisa de `nullable + backfill`, nunca `NOT NULL` seco.
- **Conflito de PR** dá pra resolver **sem apagar a autoria** de quem contribuiu (cherry-pick).

## Links

- [[RF005 - Registro de implementação (o que fizemos)]] - o cadastro de turma que os relatórios leem
- [[RF006 - Registro de implementação (o que fizemos)]] - as matrículas que o RF009 lista
- [[NF006 - Registro de implementação (o que fizemos)]] - a auditoria (também Sprint 3)
- [[Modelagem do Banco de Dados]] - as tabelas por trás dos JOINs
