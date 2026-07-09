---
title: RF010 e RF011 - Registro de implementação (o que fizemos)
tags: [rf010, rf011, noticias, editais, registro, sprint-3, backend]
status: concluído
sprint: 3
card_trello: "[RF010] Modelo de Notícias / [RF011] Modelo de Editais"
---

# RF010 e RF011 - Registro de implementação (o que fizemos)

> Status: concluído. Testado local: login admin, POST 201, GET público, POST sem token = 401.

Feitos juntos por serem gêmeos (mesmo CRUD). Veja o roteiro em [[RF010 e RF011 - Notícias e Editais (guia)]].

## O que os cards pediam

- **RF010:** tabela `Noticia` (titulo, descricao, link, imagem_url) + schemas + rotas criar/listar. `imagem_url` opcional.
- **RF011:** tabela `Edital` (titulo, descricao, link) + schemas + rotas criar/listar.

## O que fizemos (arquivos)

| Arquivo | Papel |
|---|---|
| `models/noticia_model.py` | tabela `noticias` (com `imagem_url`) |
| `models/edital_model.py` | tabela `editais` |
| `schemas/noticia_schema.py` | NoticiaCreate / NoticiaResponse (+ `example`) |
| `schemas/edital_schema.py` | EditalCreate / EditalResponse (+ `example`) |
| `routes/noticia_routes.py` | POST admin + GET público |
| `routes/edital_routes.py` | POST admin + GET público |
| `alembic/versions/b96a6baeb776_...` | migração que cria as duas tabelas |
| `main.py` / `alembic/env.py` | registro dos models + `include_router` |

## As decisões que tomamos

1. **GET público, POST admin.** O site precisa exibir notícias/editais sem login; criar é da secretaria. `POST` leva `Depends(require_admin)`, `GET` não.
2. **`imagem_url` = só a URL (texto).** Sem upload de arquivo — o card diz que a imagem já vem hospedada em nuvem. A gente só guarda o endereço.
3. **`imagem_url` opcional; resto obrigatório.** Fiel ao card.

## A migração

- Registramos os dois models no `env.py` **antes** de gerar (pra não repetir a migração vazia da NF006).
- `revision --autogenerate` detectou "added table 'noticias'" e "added table 'editais'", nada mais.
- `down_revision = 4f7bbc7b6dd0` (encadeou certinho no head da NF006 — sem o problema de cadeia que tivemos no RF004).
- `upgrade head` aplicado; banco em `b96a6baeb776`.

## O teste (resultado real)

```
1) login admin: OK
2) POST /noticias/ (admin): OK id=1
3) POST /editais/ (admin): OK id=1
4) GET /noticias/ (SEM token): OK total=1
5) POST /noticias/ SEM token: 401 (barrado — correto)
```

Depois apagamos as linhas de teste pra não sujar o banco compartilhado.

## Aprendizados

- Quando dois cards têm a mesma forma, dá pra fazer juntos — o padrão model→schema→routes já está no músculo.
- Guardar **URL** em vez de arquivo é o jeito simples e comum de lidar com imagem/documento numa API.
- A checklist de teste "cria com token / lista sem token / barra sem token" é o que prova que a regra de permissão ficou certa.

## Git
- Branch: `feature/rf010-rf011-noticias-editais`
- Commit: `fbfc96c` (9 arquivos)
- Base: `main` já com NF006 e NF001.

## Links
- [[RF010 e RF011 - Notícias e Editais (guia)]] - o roteiro
- [[Proteger Rotas - Registro de implementação (o que fizemos)]] - o `require_admin`
- Cards: [RF010] Modelo de Notícias · [RF011] Modelo de Editais
