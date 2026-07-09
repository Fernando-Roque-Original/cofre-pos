---
title: RF010 e RF011 - Notícias e Editais (guia)
tags: [rf010, rf011, noticias, editais, guia, sprint-3, backend, crud]
status: concluído
sprint: 3
---

# RF010 e RF011 - Notícias e Editais (guia)

> Dois cards, o mesmo trabalho. Notícia e Edital são o mesmo CRUD simples (título, descrição, link), então dá pra fazer os dois juntos seguindo o padrão que já repetimos 6 vezes (aluno, docente, disciplina, turma, matrícula, log).

## Por que tratamos os dois juntos

As especificações são quase idênticas:

| | Notícia (RF010) | Edital (RF011) |
|---|---|---|
| Campos | titulo, descricao, link, **imagem_url** | titulo, descricao, link |
| Schemas | NoticiaCreate / NoticiaResponse | EditalCreate / EditalResponse |
| Rotas | criar + listar | criar + listar |

A única diferença real é que a notícia tem uma imagem. Fora isso, é copiar e adaptar. Por isso vale entender **uma vez** o padrão e aplicar nos dois.

## Decisões que os cards não traziam (e por que decidimos assim)

Os cards não falavam de permissão nem de upload. A gente analisou (que nem combinamos: se o card deixa buraco, a gente resolve com bom senso):

1. **Quem pode criar? → GET público, POST só admin.**
   Notícia e edital são **conteúdo que o site mostra pra qualquer visitante** (então listar não pode exigir login), mas **criar** é trabalho da secretaria. Logo: `GET` livre, `POST` com `require_admin` — o mesmo porteiro dos outros cadastros.

2. **imagem_url guarda só a URL, não o arquivo.**
   O próprio card diz "URL pública do arquivo hospedado em nuvem (S3/Firebase)". Ou seja, o upload da imagem é feito **fora** da nossa API (pelo front ou por outro serviço), e a gente só salva o texto do endereço. Isso simplifica muito — nada de lidar com bytes de arquivo.

3. **imagem_url é opcional; título, descrição e link são obrigatórios.**
   Seguimos o card (só a imagem é marcada como opcional). Se depois quiserem deixar o `link` opcional também, é trocar uma linha no schema.

## O padrão — os 3 arquivos de cada feature

### 1. Model (a tabela)
`models/noticia_model.py` — descreve a tabela pro SQLAlchemy. `Text` no corpo porque a descrição pode ser longa; `String` no resto.

```python
class Noticia(Base):
    __tablename__ = "noticias"
    id = Column(Integer, primary_key=True, index=True)
    titulo = Column(String(255), nullable=False)
    descricao = Column(Text, nullable=False)
    link = Column(String(500), nullable=False)
    imagem_url = Column(String(500), nullable=True)   # opcional
```

O `edital_model.py` é igual, só sem o `imagem_url`.

### 2. Schema (o que entra e o que sai)
`schemas/noticia_schema.py` — o Pydantic valida a entrada e formata a saída.
- `NoticiaCreate`: o que o cliente **envia** (sem id — o banco gera). Tem `Field(min_length=...)` pra recusar dados vazios e um `example` (aquele do NF001) que aparece no `/docs`.
- `NoticiaResponse`: o que a API **devolve** (com o id). `from_attributes=True` deixa o Pydantic ler direto do objeto do banco.

### 3. Routes (os endpoints)
`routes/noticia_routes.py` — duas rotas:
- `POST /noticias/` com `usuario: dict = Depends(require_admin)` → só admin cria (201).
- `GET /noticias/` **sem** dependência de auth → qualquer um lista (mais recentes primeiro, `order_by(Noticia.id.desc())`).

## A migração (Alembic) — criar as tabelas no banco

O model é só a "planta". Pra a tabela existir de verdade no Postgres, a gente gera e aplica a migração:

1. **Registrar os models no `alembic/env.py`** (senão o autogenerate não enxerga e sai migração vazia — foi a lição da NF006):
   ```python
   import models.noticia_model
   import models.edital_model
   ```
2. **Gerar**: `alembic revision --autogenerate -m "RF010 RF011 - tabelas noticias e editais"`
   O Alembic detecta "added table 'noticias'" e "added table 'editais'".
3. **Conferir** o arquivo gerado (tem que ter os dois `create_table` e o `down_revision` apontando pro head anterior).
4. **Aplicar**: `alembic upgrade head`.

## Ligar na aplicação (`main.py`)

Três acréscimos:
```python
import models.noticia_model            # registra o model no SQLAlchemy
import models.edital_model
from routes import ..., noticia_routes, edital_routes
...
app.include_router(noticia_routes.router)
app.include_router(edital_routes.router)
```

## Como testar

Com a API no ar:
- `POST /login` (admin) → copia o `access_token`.
- `POST /noticias/` com o token → deve dar **201**.
- `GET /noticias/` **sem** token → deve dar **200** (público).
- `POST /noticias/` **sem** token → deve dar **401** (barrado).

Ver o resultado em [[RF010 e RF011 - Registro de implementação (o que fizemos)]].

## Links
- [[Modelagem do Banco de Dados]] - o padrão model → schema → routes
- [[Proteger Rotas com JWT (guia)]] - de onde vem o `require_admin` do POST
- [[NF001 - Documentação Swagger (guia)]] - o `example` que colocamos nos schemas
