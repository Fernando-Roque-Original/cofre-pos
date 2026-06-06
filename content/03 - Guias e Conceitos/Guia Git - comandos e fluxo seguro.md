---
title: Guia Git - comandos e fluxo seguro
tags: [git, github, guia, workflow, grupo-5]
---

# Guia Git — comandos e fluxo seguro

Guia pra mexer no repositório sem medo de mandar coisa errada pro `main` de todo mundo. Lê a primeira parte com calma — ela explica o que mexe no servidor e o que não mexe.

> [!danger] A regra que mais importa
> **Nada vai pro GitHub (origin) até você rodar `git push`.** Clonar, criar branch, adicionar, commitar — tudo isso é **só no seu PC**. Só o `push` manda pra nuvem. Então relaxa: dá pra errar à vontade localmente.

---

## O modelo mental: seu código vive em 4 lugares

```
[1] Working directory  →  [2] Staging  →  [3] Repo local  →  [4] Origin (GitHub)
    (você edita)            (git add)       (git commit)        (git push)
```

| Lugar | O que é | Comando que leva pra cá |
|---|---|---|
| **1. Working directory** | os arquivos que você edita no VS Code | (você editando) |
| **2. Staging** | a "fila" do que vai entrar no próximo commit | `git add` |
| **3. Repo local** | o histórico salvo no **seu PC** | `git commit` |
| **4. Origin** | o GitHub, onde todo mundo puxa | `git push` |

> [!success] O que NÃO toca o origin (pode fazer sem medo)
> `git status`, `git add`, `git commit`, `git branch`, `git checkout`, `git log`, `git diff` — **tudo local**. Nada disso some pra nuvem.

> [!warning] O que TOCA o origin
> Só dois: `git push` (envia) e `git pull` / `git fetch` (recebe). Mais nada manda código pro servidor.

---

## Comandos de inspeção (100% seguros, use sempre)

Esses só mostram informação, nunca mudam nada. Use à vontade:

```bash
git status        # o que mudou, em que branch você está
git branch        # lista as branches; a atual tem um *
git log --oneline # histórico de commits, resumido
git diff          # o que você mudou desde o último commit
```

> [!tip] Antes de qualquer coisa, roda `git status`
> Ele te diz em que branch você está e o que está pra ser commitado. É o seu "onde estou?".

---

## clone — baixar o repositório

```bash
git clone https://github.com/hiag-code/poswebreactdeploy.git
```

Baixa o projeto inteiro + todo o histórico pro seu PC. Faz **uma vez** só. Quando clona, o git já guarda o endereço do GitHub com o apelido **`origin`**.

---

## origin — o apelido do GitHub

`origin` é só um **apelido** pro endereço do repositório no GitHub. Em vez de digitar a URL toda, você fala `origin`.

```bash
git remote -v     # mostra pra onde "origin" aponta
```

Quando você faz `git push origin minha-branch`, está dizendo "manda minha-branch pro GitHub".

---

## main — a branch principal

`main` (antiga "master") é a branch **oficial** do projeto — o código que vale pra todo mundo.

> [!danger] Nunca trabalhe direto na main
> Você nunca edita a `main` direto. Você cria uma branch sua, trabalha nela, e depois pede pra juntar (via Pull Request). Assim a `main` só recebe código revisado.

---

## branch — sua linha de trabalho isolada

Uma branch é uma **cópia paralela** do código onde você trabalha sem afetar os outros. Cada tarefa = uma branch.

```bash
# criar uma branch E já ir pra ela (o jeito mais usado)
git checkout -b feature/modelagem-usuario

# trocar pra uma branch que já existe
git checkout main

# listar branches (a atual tem *)
git branch
```

> [!note] Nome de branch
> Use nomes que dizem o que é: `feature/login-jwt`, `feature/modelagem-usuario`, `fix/env-duplicado`. Evita espaço e acento.

Quando você cria uma branch, suas mudanças **não commitadas vão junto** com você pra ela. Então dá pra editar primeiro e criar a branch depois, sem perder nada.

---

## add e commit — salvar no seu PC

```bash
git add .                              # joga TUDO que mudou pra staging
git add back/models/user_model.py      # ou só um arquivo específico
git commit -m "feat: cria model Usuario"
```

- `git add` = escolhe o que entra no commit
- `git commit` = tira uma "foto" e salva no histórico **local**

> [!success] Commit é local
> Pode commitar quantas vezes quiser. Nada disso vai pro GitHub até o `push`. Commits pequenos e frequentes são bons.

**Mensagem de commit** — padrão simples:
- `feat: ...` (funcionalidade nova)
- `fix: ...` (correção de bug)
- `docs: ...` (documentação)

---

## push — o único que manda pro GitHub

```bash
# a PRIMEIRA vez que envia uma branch nova
git push -u origin feature/modelagem-usuario

# nas próximas (a branch já está ligada ao origin)
git push
```

> [!danger] NUNCA faça isto
> ```bash
> git push origin main      # manda direto pra branch oficial
> ```
> Sempre empurre pra **sua branch**, nunca pra `main`. O nome depois de `origin` tem que ser o da SUA branch.

---

## pull e fetch — receber o que mudou

Quando outra pessoa do grupo manda código pro GitHub, você puxa pro seu PC:

```bash
git pull origin main    # baixa e já junta as novidades da main
git fetch               # só baixa, sem juntar (pra você ver antes)
```

> [!tip] Antes de começar o dia, atualiza a main
> ```bash
> git checkout main
> git pull origin main
> git checkout -b feature/minha-tarefa
> ```
> Assim sua branch nasce a partir do código mais recente.

---

## fork vs clone vs branch (a confusão clássica)

| Termo | O que é | Quando usar |
|---|---|---|
| **clone** | baixar um repo pro seu PC | uma vez, pra começar |
| **branch** | linha de trabalho paralela **dentro** do repo | toda tarefa nova |
| **fork** | uma **cópia sua** do repo, na sua conta do GitHub | quando você não tem acesso de escrita ao repo original |

> [!note] No nosso caso
> Vocês têm acesso de escrita ao `hiag-code/poswebreactdeploy` (são do grupo), então **não precisam de fork** — só de **branches** dentro do repo. Fork é mais pra contribuir em projeto de estranho.

---

## Pull Request (PR) — pedir pra juntar na main

Depois que sua branch está pronta e empurrada pro GitHub:

1. Abre o repositório no GitHub
2. Aparece um botão "Compare & pull request" — clica
3. Descreve o que você fez
4. Pede pra um colega revisar
5. Depois de aprovado, alguém clica em "Merge" → aí sim entra na `main`

> [!success] Por que PR é bom
> A `main` só recebe código que passou por revisão. Ninguém quebra o projeto dos outros sem querer.

---

## Fluxo seguro do dia a dia (a receita)

Decora essa sequência:

```bash
# 1. Atualiza a main
git checkout main
git pull origin main

# 2. Cria sua branch da tarefa
git checkout -b feature/minha-tarefa

# 3. Trabalha... edita arquivos no VS Code ...

# 4. Vê o que mudou
git status
git diff

# 5. Salva (local)
git add .
git commit -m "feat: descrição do que fiz"

# 6. Manda pra SUA branch no GitHub
git push -u origin feature/minha-tarefa

# 7. Abre o PR no GitHub e pede revisão
```

---

## Regras de ouro pra não mandar pra main sem querer

> [!danger] As 5 regras
> 1. **Sempre roda `git status` antes de commitar** — confirma a branch (não pode ser `main`)
> 2. **Sempre cria uma branch** antes de mexer: `git checkout -b feature/...`
> 3. **No `push`, escreve o nome da SUA branch**, nunca `main`
> 4. **Nunca** `git push origin main` nem `git push --force`
> 5. Se na dúvida, **para e pergunta no grupo** antes de dar push

---

## Situações de pânico (e como sair)

> [!question] "Commitei na main sem querer (mas não dei push)"
> Tranquilo, ainda é local. Cria a branch a partir daqui (leva o commit junto) e volta a main pro lugar:
> ```bash
> git branch feature/salva-meu-trabalho   # salva seu commit numa branch
> git checkout feature/salva-meu-trabalho
> ```
> Depois você conserta a main com calma (ou pede ajuda no grupo).

> [!question] "Editei um monte e quero descartar tudo que NÃO commitei"
> ```bash
> git restore .          # desfaz mudanças não commitadas (cuidado: perde o que editou)
> ```

> [!question] "Quero ver em que branch estou"
> ```bash
> git branch             # a atual tem o *
> ```
> No VS Code também aparece no cantinho inferior esquerdo.

> [!question] "Dei push e quero saber se foi pra branch certa"
> Abre o repo no GitHub e olha a lista de branches. A sua deve estar lá, separada da main.

---

## Como fica no nosso projeto

- Cada tarefa do Trello = uma branch (`feature/login-jwt`, `feature/modelagem-usuario`...)
- Trabalha na branch, commita à vontade (local)
- `push` só pra sua branch
- Abre PR, colega revisa, aí junta na main
- Move o card no Trello: `em andamento` → `em revisão` (quando abre o PR) → `concluído` (quando faz merge)

---

## Links

- Documentação Git (PT-BR): https://git-scm.com/book/pt-br/v2
- Aprender Git visualmente: https://learngitbranching.js.org/?locale=pt_BR
- Notas: [[Como começar - Sprint 1]] · [[00 - LEIA PRIMEIRO - Repositórios novos do professor]]
