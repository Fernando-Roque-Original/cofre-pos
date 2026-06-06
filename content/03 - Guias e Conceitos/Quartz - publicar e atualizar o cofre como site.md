---
title: Quartz - publicar e atualizar o cofre como site
tags: [quartz, site, publicacao, ferramentas, compartilhamento]
status: em andamento
---

# Quartz - publicar e atualizar o cofre como site

> Status: site montado localmente (em andamento) - falta publicar no GitHub Pages.

## O que é o Quartz

O Quartz é um gerador de site estático feito sob medida pra vaults do Obsidian. Ele pega as notas em markdown e transforma num site navegável, mantendo:

- as pastas (viram o menu lateral),
- os wikilinks `[[...]]` (viram links clicáveis),
- o grafo, a busca, os backlinks, os callouts, o modo claro/escuro, e funciona no celular.

Por que a gente usa: pra compartilhar o cofre com o grupo **sem todo mundo precisar instalar o Obsidian**. Vira um site (só leitura) que abre por um link.

> [!info] Quartz x Obsidian
> - **Obsidian** = onde a gente ESCREVE (edita as notas).
> - **Quartz** = o que PUBLICA (mostra as notas como site, só leitura).
>
> Quem edita usa Obsidian; quem só lê abre o link do site.

## Como funciona (o fluxo)

```
Você edita no Obsidian          (pasta CofrePosGraduacao)
        ↓  sincronizar (copiar)
Quartz lê a pasta content/      (uma CÓPIA do cofre)
        ↓  build
Gera o site em public/          (HTML pronto)
        ↓  deploy (git push)
GitHub Pages publica            (link público pro grupo)
```

O ponto que mais confunde: **o Quartz NÃO lê o cofre direto**. Ele lê a pasta `content/` dele, que é uma **cópia**. Por isso, toda vez que você muda as notas, precisa **sincronizar** (copiar o cofre pra dentro do `content/`). Detalhe na seção "Como atualizar".

## Onde está e a estrutura

A pasta do Quartz: `C:\Users\ferna\Desktop\Projeto site Pos-Graduação\quartz-site`

| Pasta / arquivo | O que é | Mexe? |
|---|---|---|
| `content/` | as notas que viram o site (CÓPIA do cofre) | via sync, não na mão |
| `quartz.config.ts` | configuração (título, endereço, tema) | sim, pra configurar |
| `quartz.layout.ts` | layout das páginas (o que aparece onde) | raramente |
| `public/` | o site gerado pelo build | não (é resultado) |
| `node_modules/` | dependências instaladas | não |

## Como configurar

A configuração fica em `quartz.config.ts`. O principal:

- **pageTitle** - o título que aparece no topo do site:
```ts
pageTitle: "Cofre - Pós-Graduação IFBA",
```
- **baseUrl** - o endereço do site (preenche quando publicar no GitHub Pages):
```ts
baseUrl: "seu-usuario.github.io/nome-do-repo",
```
- **tema e cores** ficam mais abaixo, no bloco `theme` (dá pra trocar a cor de destaque, a fonte etc.).

## Como atualizar o site (sincronizar o cofre)

Sempre que editar as notas no Obsidian e quiser que o site reflita, faça o **sync**. No PowerShell:

```powershell
# 1) entrar na pasta do Quartz
cd "C:\Users\ferna\Desktop\Projeto site Pos-Graduação\quartz-site"

# 2) sincronizar: limpa o content e copia o cofre atualizado
$cofre = "C:\Users\ferna\Desktop\Projeto site Pos-Graduação\CofrePosGraduacao"
Remove-Item ".\content\*" -Recurse -Force
Get-ChildItem -LiteralPath $cofre | Where-Object { $_.Name -ne '.obsidian' -and $_.Name -ne '.trash' } | ForEach-Object { Copy-Item -LiteralPath $_.FullName -Destination ".\content" -Recurse -Force }
Copy-Item ".\content\Índice do Cofre.md" ".\content\index.md" -Force

# 3) ver o resultado localmente
npx quartz build --serve
```

Depois abre **http://localhost:8080** no navegador.

> [!note] Por que aquele `Copy-Item ... index.md`?
> O Quartz usa `content/index.md` como **página inicial** do site. Como a nossa capa é a "Índice do Cofre", a gente copia ela pra virar a `index.md`. Sem isso, a home fica sem página de entrada.

> [!warning] Não edite dentro de content/
> O `content/` é uma cópia e é **sobrescrito** a cada sync. Edite SEMPRE no cofre (Obsidian). Se mexer direto no `content/`, perde na próxima sincronização.

## Comandos do dia a dia

| Quero... | Comando (dentro de `quartz-site`) |
|---|---|
| Ver o site local (recarrega sozinho ao salvar) | `npx quartz build --serve` -> http://localhost:8080 |
| Só gerar o site (sem servir) | `npx quartz build` |
| Parar o preview | `Ctrl + C` no terminal |

## Publicar pro grupo (deploy) - próximo passo

Pra o grupo acessar por um link, o site precisa ir pro **GitHub Pages** (grátis). Resumo (a gente detalha quando for fazer):

1. Criar um repositório no GitHub só pro site.
2. Preencher o `baseUrl` e configurar o workflow de deploy (GitHub Action que já vem no Quartz).
3. `npx quartz sync` envia o conteúdo; o GitHub Action builda e publica.
4. Daí em diante, atualizar = sync do cofre + `npx quartz sync` -> o site se atualiza sozinho.

> [!tip] Versão do Quartz
> A gente usa a **v4** (estável). A v5 (branch `main`) é uma reescrita nova que deu erro de build na montagem, por isso ficamos na v4.

> [!info] Quer esconder alguma pasta do site?
> Dá pra não publicar uma pasta (ex: "Documentos Fonte") tirando ela do sync (adicionando o nome no filtro `Where-Object`) ou usando o `ignorePatterns` no `quartz.config.ts`.

## Links

- [[Índice do Cofre]]
- [[Tecnologias e Bibliotecas do Projeto]]
- [[Guia Git - comandos e fluxo seguro]] - pro passo de publicar (git)
- Site oficial do Quartz: quartz.jzhao.xyz
