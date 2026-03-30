# Car API

Projeto desenvolvido durante o curso de **FastAPI** da **PycodeBR**.

Esta API foi construída para praticar conceitos de desenvolvimento backend com FastAPI, autenticação com JWT, modelagem de dados com SQLAlchemy, migrações com Alembic e testes automatizados.

## Visao geral

A aplicacao expoe uma API REST para gerenciamento de:

- usuarios
- autenticacao
- marcas
- carros

Tambem conta com documentacao tecnica em `docs/` e documentacao interativa automatica via Swagger e ReDoc.

## Stack utilizada

### Backend

- Python 3.13
- FastAPI
- Pydantic v2

### Banco de dados

- SQLAlchemy 2 com suporte assincrono
- SQLite com `aiosqlite`
- Alembic para migracoes
- `psycopg[binary]` ja configurado como dependencia para uso com PostgreSQL, se necessario

### Autenticacao e seguranca

- JWT com `PyJWT`
- Hash de senhas com `pwdlib[argon2]`

### Qualidade e produtividade

- Pytest
- pytest-asyncio
- pytest-cov
- Ruff
- Taskipy
- Poetry

### Documentacao

- MkDocs
- MkDocs Material
- pymdown-extensions

## Funcionalidades

- cadastro e listagem de usuarios
- autenticacao com geracao de token JWT
- renovacao de token
- CRUD de marcas
- CRUD de carros
- endpoint de health check
- validacao de dados com Pydantic
- migracoes de banco com Alembic
- testes automatizados

## Estrutura do projeto

```text
car_api/
├── car_api/
│   ├── app.py
│   ├── core/
│   ├── models/
│   ├── routers/
│   └── schemas/
├── docs/
├── migrations/
├── tests/
├── pyproject.toml
├── alembic.ini
├── mkdocs.yml
└── README.md
```

### Principais diretorios

- `car_api/app.py`: ponto de entrada da aplicacao
- `car_api/core/`: configuracoes, banco e seguranca
- `car_api/models/`: modelos ORM
- `car_api/schemas/`: schemas de validacao e resposta
- `car_api/routers/`: rotas da API
- `migrations/`: historico de migracoes do banco
- `tests/`: testes automatizados
- `docs/`: documentacao complementar do projeto

## Endpoints principais

### Saude da aplicacao

- `GET /health_check`

### Autenticacao

- `POST /api/v1/auth/token`
- `POST /api/v1/auth/refresh_token`

### Usuarios

- `POST /api/v1/users/`
- `GET /api/v1/users/`
- `GET /api/v1/users/{user_id}`
- `PUT /api/v1/users/{user_id}`
- `DELETE /api/v1/users/{user_id}`

### Marcas

- `POST /api/v1/brands/`
- `GET /api/v1/brands/`
- `GET /api/v1/brands/{brand_id}`
- `PUT /api/v1/brands/{brand_id}`
- `DELETE /api/v1/brands/{brand_id}`

### Carros

- `POST /api/v1/cars/`
- `GET /api/v1/cars/`
- `GET /api/v1/cars/{car_id}`
- `PUT /api/v1/cars/{car_id}`
- `DELETE /api/v1/cars/{car_id}`

## Configuracao do ambiente

O projeto usa variaveis de ambiente carregadas pelo arquivo `.env`.

Exemplo:

```env
DATABASE_URL='sqlite+aiosqlite:///car.db'
JWT_SECRET_KEY='sua-chave-secreta'
JWT_ALGORITHM='HS256'
JWT_EXPIRATION_MINUTES=30
```

## Como instalar

### 1. Instalar dependencias

```bash
poetry install --with dev
```

### 2. Aplicar migracoes

```bash
poetry run alembic upgrade head
```

### 3. Rodar a aplicacao

```bash
poetry run task run
```

Ou diretamente:

```bash
poetry run fastapi dev car_api/app.py
```

## Como testar

```bash
poetry run task test
```

Ou com pytest diretamente:

```bash
poetry run pytest -s -x --cov=car_api -vv
```

## Como gerar a documentacao

```bash
poetry run task docs
```

## Acessos locais

Com a aplicacao em execucao:

- API: `http://127.0.0.1:8000`
- Swagger UI: `http://127.0.0.1:8000/docs`
- Health check: `http://127.0.0.1:8000/health_check`
- MkDocs: `http://127.0.0.1:8001`

## Comandos uteis

```bash
poetry run task run
poetry run task test
poetry run task lint
poetry run task format
poetry run task docs
```

## Observacao

Este projeto faz parte dos estudos realizados no curso de **FastAPI da PycodeBR**, servindo como base pratica para aprender organizacao de APIs, autenticacao, banco de dados, testes e documentacao em Python.
