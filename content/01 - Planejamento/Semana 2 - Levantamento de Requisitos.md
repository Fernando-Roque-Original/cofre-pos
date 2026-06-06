---
title: Levantamento de Requisitos — Semana 2
tags: [requisitos, semana-2, grupo-5, fastapi, pos-graduacao]
created: 2026-05-20
prazo: 2026-05-17
status: pendente
responsavel: Grupo 5
---

# Levantamento de Requisitos — Semana 2 (11–17/mai)

> **Objetivo desta semana:** Produzir o Documento de Requisitos completo para guiar o desenvolvimento das próximas 11 semanas.
> **Entrega:** até 17/05/2026
> **Baseado em:** Descrição do Desafio 5 + TCCs anteriores (Bem-Hur e Letícia) + repositório pos-web

---

## Checklist da semana

- [ ] Preencher todas as seções deste documento
- [ ] Validar campos das entidades com o orientador (Crescêncio)
- [ ] Rascunhar o DER (Diagrama Entidade-Relacionamento)
- [ ] Confirmar regras de negócio com a coordenação do PGDW
- [ ] Identificar e documentar o formato dos CSVs existentes
- [ ] Listar os relatórios esperados
- [ ] Revisar com o grupo e fechar o documento

---

## 1. Objetivo do sistema

> *Descreva em 2–3 frases o que o sistema faz e para quem.*

O sistema é um **back-end web para gestão acadêmica da pós-graduação do IFBA**, desenvolvido com FastAPI. Ele centraliza o controle de alunos, professores, disciplinas, turmas, matrículas e histórico acadêmico, disponibilizando uma API REST documentada automaticamente via Swagger/OpenAPI.

**Usuários principais do sistema:**
- [ ] Administrador (coordenação)
- [ ] Professor
- [ ] Aluno
- [ ] Outro: ___________

---

## 2. Entidades e campos

> *Para cada entidade, confirme os campos, tipos e se são obrigatórios. Marque com os confirmados pelo grupo.*

---

### 2.1 Aluno

| Campo | Tipo | Obrigatório? | Observações |
|---|---|---|---|
| id | inteiro (auto) | | Gerado pelo banco |
| matricula | texto | | Formato? Ex: `2024001` |
| nome_completo | texto | | |
| email | texto | | Único no sistema? |
| cpf | texto | | Validar formato? Único? |
| telefone | texto | | |
| data_nascimento | data | | |
| endereco | texto | | Endereço completo ou campos separados? |
| curso | texto | | PGDW fixo ou selecionável? |
| periodo_ingresso | texto | | Ex: `2024.1` |
| situacao | texto | | Ativo / Trancado / Concluído / Desligado |
| criado_em | data/hora | | Automático |

**Dúvidas a confirmar com o orientador:**
- [ ] A matrícula é gerada pelo sistema ou vem de fora (SUAP, SIGAA)?
- [ ] O aluno tem acesso ao sistema com login próprio?
- [ ] Precisa armazenar lattes, currículo ou documentos?

---

### 2.2 Professor

| Campo | Tipo | Obrigatório? | Observações |
|---|---|---|---|
| id | inteiro (auto) | | |
| nome_completo | texto | | |
| email | texto | | Único |
| cpf | texto | | |
| titulacao | texto | | Especialista / Mestre / Doutor |
| areas_atuacao | texto | | Ex: "Engenharia de Software, IA" |
| vinculo | texto | | Efetivo / Substituto / Voluntário |
| lattes | texto (URL) | | Link do currículo Lattes |
| ativo | booleano | | Padrão: verdadeiro |
| criado_em | data/hora | | |

**Dúvidas a confirmar:**
- [ ] Professor pode ministrar mais de uma disciplina por semestre?
- [ ] Professor tem login para lançar notas diretamente no sistema?
- [ ] Existe professor externo (fora do IFBA)?

---

### 2.3 Disciplina

| Campo | Tipo | Obrigatório? | Observações |
|---|---|---|---|
| id | inteiro (auto) | | |
| codigo | texto | | Ex: `PGDW001`. Único? |
| nome | texto | | |
| ementa | texto longo | | Descrição do conteúdo |
| carga_horaria | inteiro | | Em horas. Ex: 60 |
| creditos | inteiro | | Ex: 4 créditos |
| tipo | texto | | Obrigatória / Eletiva / Optativa |
| pre_requisitos | lista | | IDs de outras disciplinas |
| ativa | booleano | | Padrão: verdadeiro |

**Dúvidas a confirmar:**
- [ ] Existe pré-requisito entre disciplinas? É verificado automaticamente?
- [ ] Uma disciplina pode ser oferecida em mais de um semestre simultaneamente?
- [ ] Tem equivalência entre disciplinas?

---

### 2.4 Turma

| Campo | Tipo | Obrigatório? | Observações |
|---|---|---|---|
| id | inteiro (auto) | | |
| disciplina_id | inteiro (FK) | | Referência à disciplina |
| professor_id | inteiro (FK) | | Referência ao professor |
| semestre | texto | | Ex: `2024.1` |
| vagas_total | inteiro | | Máximo de alunos |
| vagas_disponíveis | inteiro | | Calculado automaticamente |
| horario | texto | | Ex: `Sáb 08:00–12:00` |
| sala | texto | | Presencial? Online? Híbrido? |
| modalidade | texto | | Presencial / EAD / Híbrido |
| status | texto | | Aberta / Em andamento / Encerrada |

**Dúvidas a confirmar:**
- [ ] Qual o limite máximo de alunos por turma no PGDW?
- [ ] Uma turma pode ter mais de um professor?
- [ ] Turma pode ser cancelada depois de aberta?

---

### 2.5 Matrícula

| Campo | Tipo | Obrigatório? | Observações |
|---|---|---|---|
| id | inteiro (auto) | | |
| aluno_id | inteiro (FK) | | |
| turma_id | inteiro (FK) | | |
| data_matricula | data | | Automático no momento da matrícula |
| status | texto | | Ativa / Trancada / Cancelada / Concluída |
| nota_final | decimal | | Ex: 7.5 |
| frequencia | decimal | | Ex: 85.0 (%) |
| situacao_final | texto | | Aprovado / Reprovado / Trancado |
| data_conclusao | data | | Quando a disciplina foi concluída |

**Dúvidas a confirmar:**
- [ ] Qual a nota mínima para aprovação? (ex: 7.0?)
- [ ] Qual a frequência mínima? (ex: 75%?)
- [ ] Aluno pode se matricular em quantas disciplinas por semestre?
- [ ] Existe prazo para trancamento de matrícula?
- [ ] Matrícula cancelada pelo admin é diferente de trancamento pelo aluno?

---

### 2.6 Histórico Acadêmico

> *O histórico é gerado a partir das matrículas concluídas. Confirme o que precisa ser calculado.*

| Informação | Gerado automaticamente? | Observações |
|---|---|---|
| Disciplinas cursadas | | Lista de todas as matrículas do aluno |
| IRA (Índice de Rendimento Acadêmico) | | Média ponderada de todas as notas |
| Total de créditos concluídos | | Soma dos créditos das disciplinas aprovadas |
| Situação geral | | Regular / Irregular / Concluído |
| Coeficiente de rendimento | | Igual ao IRA ou diferente? |

**Dúvidas a confirmar:**
- [ ] Qual a fórmula do IRA usada pelo IFBA?
- [ ] O histórico pode ser exportado em PDF?
- [ ] Reprovações aparecem no histórico?

---

### 2.7 Usuário (autenticação e permissões)

| Campo | Tipo | Obrigatório? | Observações |
|---|---|---|---|
| id | inteiro (auto) | | |
| email | texto | | Login do sistema |
| senha_hash | texto | | Nunca salvar senha pura |
| perfil | texto | | admin / professor / aluno |
| ativo | booleano | | Padrão: verdadeiro |
| aluno_id | inteiro (FK) | | Se perfil = aluno |
| professor_id | inteiro (FK) | | Se perfil = professor |
| ultimo_acesso | data/hora | | |

**Permissões por perfil — confirmar com o orientador:**

| Ação | Admin | Professor | Aluno |
|---|---|---|---|
| Cadastrar alunos | | | |
| Editar alunos | | | |
| Cadastrar professores | | | |
| Criar disciplinas | | | |
| Criar turmas | | | |
| Matricular aluno em turma | | | |
| Lançar nota | | | |
| Ver histórico próprio | | | |
| Ver histórico de qualquer aluno | | | |
| Gerar relatórios | | | |
| Importar CSV | | | |

---

## 3. Regras de negócio

> *Liste todas as regras que o sistema deve aplicar automaticamente. Numeradas para referência no código.*

### 3.1 Matrícula
- **RN01:** O sistema deve bloquear matrícula se não houver vagas disponíveis na turma.
- **RN02:** O sistema deve bloquear matrícula duplicada (mesmo aluno na mesma turma).
- **RN03:** O sistema verifica pré-requisitos antes de matricular? — *A confirmar*
- **RN04:** Limite máximo de disciplinas por aluno por semestre = _____ — *A confirmar*
- **RN05:** Existe período de matrícula (datas de abertura e fechamento)? — *A confirmar*

### 3.2 Aprovação e reprovação
- **RN06:** Nota mínima para aprovação = _____ — *A confirmar*
- **RN07:** Frequência mínima para aprovação = _____% — *A confirmar*
- **RN08:** Reprovado por falta e reprovado por nota são situações distintas? Sim Não
- **RN09:** Aluno reprovado pode refazer a disciplina? Tem limite de tentativas?

### 3.3 Trancamento
- **RN10:** Trancamento de matrícula é por disciplina ou do curso inteiro?
- **RN11:** Prazo máximo para trancamento: _____ — *A confirmar*
- **RN12:** Trancamento conta como tentativa de aprovação?

### 3.4 Turmas
- **RN13:** Uma turma só pode ser aberta se tiver professor vinculado? Sim Não
- **RN14:** Turma pode ser cancelada com alunos matriculados? O que acontece com eles?
- **RN15:** Mínimo de alunos para a turma não ser cancelada = _____ — *A confirmar*

### 3.5 Usuários
- **RN16:** Senha deve ter no mínimo _____ caracteres.
- **RN17:** Token JWT expira em _____ horas.
- **RN18:** Existe recuperação de senha por e-mail?

---

## 4. Importação de CSV

> *O desafio exige importação de dados via CSV. Levantar o formato exato dos arquivos que o IFBA já usa.*

### 4.1 CSV de Alunos

**Campos esperados no CSV:**

| Coluna no CSV | Campo no sistema | Obrigatório? |
|---|---|---|
| (preencher) | (preencher) | |
| | | |

**Comportamento na importação:**
- [ ] Se CPF já existir: Atualiza o registro / Rejeita a linha / Pergunta ao usuário
- [ ] Se linha tiver campo inválido: Pula a linha / Para a importação / Salva parcial
- [ ] Retornar relatório de erros após importação? Sim Não

### 4.2 CSV de Professores

**Campos esperados no CSV:**

| Coluna no CSV | Campo no sistema | Obrigatório? |
|---|---|---|
| (preencher) | (preencher) | |
| | | |

---

## 5. Relatórios esperados

> *Definir os relatórios que o sistema deve gerar. Cada relatório vira um endpoint da API.*

| # | Nome do relatório | Filtros disponíveis | Formato de saída | Quem acessa |
|---|---|---|---|---|
| R01 | Alunos matriculados por turma | turma_id, semestre | JSON / PDF | Admin, Professor |
| R02 | Situação acadêmica dos alunos | semestre, situacao | JSON | Admin |
| R03 | Ocupação de turmas | semestre, disciplina | JSON | Admin |
| R04 | Histórico acadêmico do aluno | aluno_id | JSON / PDF | Admin, Aluno |
| R05 | Matrículas por semestre | semestre | JSON | Admin |
| R06 | *(adicionar)* | | | |
| R07 | *(adicionar)* | | | |

---

## 6. Integrações externas

> *O sistema precisa se comunicar com algum sistema externo?*

| Sistema | Tipo de integração | Necessário? | Observações |
|---|---|---|---|
| SUAP (IFBA) | API REST | Sim Não | Importar dados de alunos/professores? |
| SIGAA | API REST | Sim Não | |
| E-mail (SMTP) | Envio de notificações | Sim Não | Notificar matrícula, nota lançada, etc. |
| Frontend pos-web | REST + JWT | Sim | Integração principal |
| Firebase (legado) | Migração | Sim Não | Importar dados existentes? |

---

## 7. Requisitos não funcionais

> *O que o sistema deve garantir além das funcionalidades.*

| Requisito | Especificação | Confirmado? |
|---|---|---|
| Documentação automática | Swagger em `/docs` e `/redoc` | (exigido pelo desafio) |
| Autenticação | JWT com expiração | |
| Segurança | Senhas com bcrypt hash | |
| Performance | Listagens com paginação (limit/offset) | |
| CORS | Aceitar origem do frontend Vercel | |
| Deploy | Ambiente de produção acessível via URL | |
| Logs | Registrar operações críticas (login, matrícula, nota) | |
| Backup | Estratégia de backup do banco | |

---

## 8. Esboço do DER (Diagrama Entidade-Relacionamento)

> *Rascunhe aqui ou anexe a imagem do diagrama.*

```
USUARIO (1) -------- (0..1) ALUNO
USUARIO (1) -------- (0..1) PROFESSOR

ALUNO (1) ---------- (N) MATRICULA
TURMA (1) ---------- (N) MATRICULA

TURMA (N) ---------- (1) DISCIPLINA
TURMA (N) ---------- (1) PROFESSOR

DISCIPLINA (N) ----- (N) DISCIPLINA  [auto-relacionamento: pré-requisitos]
```

**Ferramenta sugerida para desenhar:** https://dbdiagram.io

---

## 9. Perguntas para o orientador (Crescêncio)

> *Dúvidas que precisam ser respondidas antes de fechar os requisitos. Levar para a Reunião 1 (07/jun) ou tirar por e-mail antes.*

1. O sistema deve importar dados do SUAP ou SIGAA, ou começa do zero?
2. Qual a fórmula exata do IRA usada pelo PGDW/IFBA?
3. Existe um regulamento acadêmico do PGDW que define as regras de matrícula/trancamento?
4. O frontend (pos-web) vai consumir esta API ou são sistemas separados por enquanto?
5. O deploy precisa ser em infraestrutura do IFBA ou pode ser em serviço externo (Railway/Render)?
6. Existe um CSV ou planilha com dados reais de alunos/professores para usar nos testes?
7. O sistema precisa de notificações por e-mail (matrícula confirmada, nota lançada)?
8. *(adicionar mais perguntas aqui)*

---

## 10. Decisões tomadas pelo grupo

> *Registre aqui as decisões que o grupo tomou durante a semana, com data e responsável.*

| Data | Decisão | Responsável |
|---|---|---|
| ___/___/2026 | | |
| ___/___/2026 | | |

---

## Referências

- [[Plano de Implementação - Desafio 5]] — plano geral do projeto
- Repositório frontend: https://github.com/crescenciolima/pos-web/tree/main
- TCC Bem-Hur: sistema de processo seletivo PGDW (Next.js + Firebase)
- TCC Letícia: sistema de gerenciamento de processos seletivos PGDW
- FastAPI docs: https://fastapi.tiangolo.com/
- dbdiagram.io (DER online): https://dbdiagram.io
