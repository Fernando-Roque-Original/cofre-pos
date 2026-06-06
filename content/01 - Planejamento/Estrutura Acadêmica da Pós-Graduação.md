---
title: Estrutura Acadêmica da Pós-Graduação
tags: [estudo, estrutura-academica, dominio, documentacao, sprint-1, grupo-5]
card_trello: Estudo da estrutura academica da Pos-Graduacao
---

# Estrutura Acadêmica da Pós-Graduação

Estudo de como a pós-graduação funciona academicamente: quais são as peças (entidades), como elas se ligam e quais as regras. É a base que usamos pra modelar o sistema.

> Este estudo saiu de duas fontes: o backend de referência do professor (os arquivos JSON com os dados reais) e o levantamento de requisitos. Detalhe técnico da modelagem em [[Modelagem do Banco de Dados]]; regras em [[Semana 2 - Levantamento de Requisitos]].

---

## Visão geral (a estrutura em uma frase)

> Um **aluno** se **matricula** numa **turma**, que é uma **disciplina** ofertada num semestre por um **docente**. Cada pessoa (aluno ou docente) tem um **usuário** (login) pra acessar o sistema.

Essa frase resume todo o domínio. As seções abaixo destrincham cada peça.

---

## As entidades (as peças do sistema)

### Usuário
Quem **acessa** o sistema. Guarda o login (email + senha) e o **perfil** (role): estudante, docente ou admin. Não é "uma pessoa acadêmica" em si - é só a credencial de acesso. Um aluno e um docente são, cada um, um usuário com um perfil diferente.

### Aluno
O **estudante** da pós. Guarda os dados acadêmicos dele: matrícula, nome, CPF, email, situação. Está ligado a um usuário (o login dele).

### Docente
O **professor** da pós. Guarda nome, CPF, email, **titulação** (mestre, doutor) e situação. Também ligado a um usuário.

### Disciplina
A **matéria** em si (ex: "Programação para Web"). Guarda nome, ementa (o conteúdo), carga horária. Existe independente de quem dá ou quando.

### Turma
A **oferta** de uma disciplina num **semestre específico**, por um **docente específico**. É o "encontro" entre disciplina + docente + semestre. Tem um número de vagas.
- Ex: a disciplina "Prog Web" virou a turma "Prog Web - 2026.1, Prof. Carlos, 30 vagas".

### Matrícula
O **vínculo** de um aluno com uma turma. É quando o aluno "entra" numa turma. Guarda a data, a situação (ativa/trancada/concluída) e a nota final.

---

## Como as entidades se relacionam

```
USUÁRIO  (login)
   │
   ├──< ALUNO ──────< MATRÍCULA >────── TURMA >──┬── DISCIPLINA
   │                                             │
   └──< DOCENTE ─────────────────────────────────┘
```

Lendo as ligações:
- Um **usuário** pode ser um aluno OU um docente (tem o login dele)
- Um **aluno** faz várias **matrículas** (uma por turma que cursa)
- Uma **turma** recebe várias **matrículas** (vários alunos)
- Uma **turma** é de **uma** disciplina e tem **um** docente
- Uma **disciplina** pode virar **várias** turmas (em semestres diferentes)
- Um **docente** ministra **várias** turmas

O detalhe técnico (chaves estrangeiras) está em [[Modelagem do Banco de Dados]].

---

## Os fluxos acadêmicos

### A jornada de um aluno
```
1. É cadastrado no sistema (vira um aluno + um login)
2. Se matricula nas turmas do semestre (as disciplinas que vai cursar)
3. Cursa, recebe nota e frequência
4. É aprovado ou reprovado em cada turma
5. Acumula disciplinas cursadas (o histórico)
```

### O papel de um docente
```
1. É cadastrado no sistema (vira um docente + um login)
2. É designado pra ministrar turmas (disciplinas em um semestre)
3. Lança notas dos alunos das suas turmas
```

### O administrador
Gerencia tudo: cadastra alunos/docentes, cria disciplinas, abre turmas, e tem acesso aos relatórios.

---

## As regras de negócio (o que o sistema deve garantir)

Algumas já estão definidas, outras são decisões a confirmar com o grupo/professor:

| Regra | Status |
|---|---|
| Uma turma tem vagas limitadas (não pode matricular além do limite) | a implementar |
| Um aluno não pode se matricular duas vezes na mesma turma | a implementar |
| Pré-requisitos entre disciplinas (precisa ter feito X antes de Y) | a confirmar |
| Nota mínima e frequência mínima pra aprovação | a confirmar (ex: 7.0 / 75%) |
| Limite de disciplinas por aluno por semestre | a confirmar |
| Trancamento de matrícula (com prazo) | a confirmar |
| CPF e email únicos (não repetem) | implementado (cadastros) |
| Senha sempre guardada em hash | implementado |

(detalhe completo das regras em [[Semana 2 - Levantamento de Requisitos]])

---

## Os perfis de acesso (roles)

| Perfil | Quem é | O que faz (resumo) |
|---|---|---|
| estudante | o aluno | vê suas matrículas, histórico, faz inscrição |
| docente | o professor | vê suas turmas, lança notas |
| admin | a coordenação | gerencia tudo (cadastros, turmas, relatórios) |

O perfil fica no campo `role` do usuário, e vai dentro do token JWT no login - é o que vai permitir, na Sprint 2, proteger as rotas por perfil.

---

## De onde veio este estudo

- **Backend de referência do professor** (https://github.com/DjanInfo/poswebserver): os arquivos JSON (alunos, docentes, disciplinas, etc.) mostraram as entidades reais e o formato dos dados.
- **Levantamento de requisitos**: as regras de negócio e o fluxo.
- **Modelagem**: ao transformar tudo isso em tabelas relacionais, a estrutura ficou formalizada.

---

## Links

- [[Modelagem do Banco de Dados]] - a estrutura em forma de banco (entidades, FKs, DER)
- [[Semana 2 - Levantamento de Requisitos]] - as regras de negócio detalhadas
- [[Plano de Implementação - Desafio 5]] - o plano geral
- Backend de referência: https://github.com/DjanInfo/poswebserver
