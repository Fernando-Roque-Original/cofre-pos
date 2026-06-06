# Cofre — Pós-Graduação Web Back-end (Grupo 5)

Site de documentação do nosso projeto da pós-graduação em Sistemas Web Back-end (IFBA). Reúne o planejamento, a modelagem do banco de dados, os registros de implementação (login com JWT, cadastros, proteção de rotas) e os guias de conceitos do backend feito em FastAPI.

## Acessar o site

**https://fernando-roque-original.github.io/cofre-pos/**

## O que tem aqui

- **00 - Comece Aqui** — contexto geral, repositórios e plano de implementação
- **01 - Planejamento** — requisitos, estrutura acadêmica e início da Sprint 1
- **02 - Implementação** — o que já foi construído, com explicação: login com JWT, cadastro de aluno e docente, e a proteção de rotas
- **03 - Guias e Conceitos** — FastAPI, SQLAlchemy, estrutura de pastas, fluxo de git e os conceitos do código

## Como funciona

As anotações são escritas no Obsidian (vault "CofrePosGraduacao") e publicadas como site usando o [Quartz](https://quartz.jzhao.xyz). A cada `push` na branch `main`, o GitHub Actions builda e atualiza o site sozinho.

- Conteúdo do site: pasta `content/`
- Configuração (título, cores, endereço): `quartz.config.ts`
- Publicação automática: `.github/workflows/deploy.yml` (GitHub Pages)

## Rodar localmente

```bash
npm install
npx quartz build --serve
# abre em http://localhost:8080
```

## Atualizar o site

1. Editar as notas no Obsidian
2. Copiar o cofre para a pasta `content/`
3. `git add . && git commit -m "atualiza notas" && git push`
4. O GitHub publica a nova versão em 1-2 minutos

---

Feito com Quartz · Grupo 5 — IFBA
