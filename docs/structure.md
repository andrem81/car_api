# Estrutura do Projeto

Este documento descreve a organização de diretórios e arquivos da Fastcar API.

## 📁 Árvore de Diretórios

```
car_api/
├── .env                          # Variáveis de ambiente
├── .gitignore                    # Arquivos ignorados pelo Git
├── .ruff_cache/                  # Cache do Ruff (linting)
├── alembic.ini                   # Configuração do Alembic
├── car.db                        # Banco de dados SQLite
├── mkdocs.yml                    # Configuração do MkDocs
├── poetry.lock                   # Lock de dependências do Poetry
├── pyproject.toml                # Configuração do projeto e dependências
├── README.md                     # README principal
├── docs/                         # Documentação do projeto
│   ├── index.md                  # Home da documentação
│   ├── overview.md               # Visão geral
│   ├── prerequisites.md          # Pré-requisitos
│   ├── installation.md           # Instalação
│   ├── configuration.md          # Configuração
│   ├── guidelines.md             # Guidelines e padrões
│   ├── structure.md              # Estrutura do projeto
│   ├── api-endpoints.md          # Endpoints da API
│   ├── system-modeling.md        # Modelagem do sistema
│   ├── authentication.md         # Autenticação e segurança
│   ├── development.md            # Desenvolvimento
│   ├── testing.md                # Testes
│   ├── deployment.md             # Deploy
│   ├── contributing.md           # Contribuição
│   └── release-notes.md          # Release notes
├── car_api/                      # Pacote principal da aplicação
│   ├── __init__.py               # Inicializador do pacote
│   ├── app.py                    # Aplicação FastAPI principal
│   ├── core/                     # Módulos centrais
│   │   ├── __init__.py
│   │   ├── database.py           # Configuração e conexão com banco
│   │   ├── security.py           # Funções de segurança e autenticação
│   │   └── settings.py           # Configurações e variáveis de ambiente
│   ├── models/                   # Modelos SQLAlchemy (ORM)
│   │   ├── __init__.py
│   │   ├── base.py               # Classe base para modelos
│   │   ├── users.py              # Modelo de Usuário
│   │   └── cars.py               # Modelos de Carro e Marca
│   ├── routers/                  # Routers da API (endpoints)
│   │   ├── __init__.py
│   │   ├── auth.py               # Endpoints de autenticação
│   │   ├── users.py              # Endpoints de usuários
│   │   ├── cars.py               # Endpoints de carros
│   │   └── brands.py             # Endpoints de marcas
│   └── schemas/                  # Schemas Pydantic (validação)
│       ├── __init__.py
│       ├── auth.py               # Schemas de autenticação
│       ├── users.py              # Schemas de usuários
│       ├── cars.py               # Schemas de carros
│       └── brands.py             # Schemas de marcas
├── migrations/                   # Migrações do Alembic
│   ├── README
│   ├── env.py                    # Ambiente de migração
│   ├── script.py.mako            # Template para migrações
│   └── versions/                 # Versões de migração
│       └── *.py                  # Scripts de migração
└── tests/                        # Testes automatizados
    ├── __init__.py
    └── *.py                      # Arquivos de teste
```

## 📄 Descrição dos Arquivos Principais

### Raiz do Projeto

| Arquivo | Descrição |
|---------|-----------|
| `pyproject.toml` | Configuração do projeto, dependências e scripts |
| `poetry.lock` | Lock file das dependências (gerado automaticamente) |
| `.env` | Variáveis de ambiente (não versionado) |
| `alembic.ini` | Configuração do Alembic para migrações |
| `mkdocs.yml` | Configuração da documentação MkDocs |
| `README.md` | Informações gerais do projeto |
| `car.db` | Banco de dados SQLite (gerado automaticamente) |

### car_api/app.py

Ponto de entrada da aplicação FastAPI:

```python
from fastapi import FastAPI, status
from car_api.routers import auth, brands, cars, users

app = FastAPI()

app.include_router(router=auth.router, prefix='/api/v1/auth', tags=['authentication'])
app.include_router(router=users.router, prefix='/api/v1/users', tags=['users'])
app.include_router(router=brands.router, prefix='/api/v1/brands', tags=['brands'])
app.include_router(router=cars.router, prefix='/api/v1/cars', tags=['cars'])

@app.get('/health_check', status_code=status.HTTP_200_OK)
def health_check():
    return {'status': 'ok'}
```

### car_api/core/

Módulos centrais da aplicação:

| Arquivo | Responsabilidade |
|---------|------------------|
| `database.py` | Configuração do engine e sessão do banco de dados |
| `security.py` | Hash de senha, JWT, autenticação, validação de ownership |
| `settings.py` | Classe Settings para gerenciamento de configurações |

#### database.py

```python
from sqlalchemy.ext.asyncio import AsyncSession, create_async_engine
from car_api.core.settings import Settings

engine = create_async_engine(Settings().DATABASE_URL)

async def get_session():
    async with AsyncSession(engine, expire_on_commit=False) as session:
        yield session
```

#### security.py

```python
# Funções principais:
- get_password_hash()      # Hash de senha com Argon2
- verify_password()        # Verificação de senha
- create_access_token()    # Criação de token JWT
- verify_token()           # Validação de token
- authenticate_user()      # Autenticação de usuário
- get_current_user()       # Obter usuário atual do token
- verify_car_ownership()   # Verificar propriedade do carro
```

#### settings.py

```python
from pydantic_settings import BaseSettings, SettingsConfigDict

class Settings(BaseSettings):
    model_config = SettingsConfigDict(
        env_file='.env', env_file_encoding='utf-8'
    )
    
    DATABASE_URL: str
    JWT_SECRET_KEY: str
    JWT_ALGORITHM: str = 'HS256'
    JWT_EXPIRATION_MINUTES: int = 30
```

### car_api/models/

Modelos SQLAlchemy que representam as tabelas do banco:

| Arquivo | Modelos |
|---------|---------|
| `base.py` | Classe base `Base` para todos os modelos |
| `users.py` | Modelo `User` |
| `cars.py` | Modelos `Brand`, `Car`, `FuelType`, `TransmissionType` |

#### Relacionamentos

```
User (1) ──────< (N) Car
Brand (1) ──────< (N) Car
```

### car_api/routers/

Endpoints da API organizados por domínio:

| Arquivo | Endpoints | Prefixo |
|---------|-----------|---------|
| `auth.py` | POST /token, POST /refresh_token | `/api/v1/auth` |
| `users.py` | CRUD completo | `/api/v1/users` |
| `cars.py` | CRUD completo com filtros | `/api/v1/cars` |
| `brands.py` | CRUD completo | `/api/v1/brands` |

### car_api/schemas/

Schemas Pydantic para validação e serialização:

| Arquivo | Schemas |
|---------|---------|
| `auth.py` | `Token`, `LoginRequest` |
| `users.py` | `UserSchema`, `UserUpdateSchema`, `UserPublicSchema`, `UserListPublicSchema` |
| `cars.py` | `CarSchema`, `CarUpdateSchema`, `CarPublicSchema`, `CarListPublicSchema` |
| `brands.py` | `BrandSchema`, `BrandUpdateSchema`, `BrandPublicSchema`, `BrandListPublicSchema` |

### migrations/

Scripts de migração do banco de dados:

| Arquivo | Descrição |
|---------|-----------|
| `env.py` | Configuração do ambiente de migração |
| `script.py.mako` | Template para novas migrações |
| `versions/` | Scripts de migração versionados |

### tests/

Testes automatizados:

```
tests/
├── __init__.py
├── test_auth.py          # Testes de autenticação
├── test_users.py         # Testes de usuários
├── test_cars.py          # Testes de carros
├── test_brands.py        # Testes de marcas
└── conftest.py           # Configurações e fixtures dos testes
```

## 📊 Organização por Camadas

```
┌─────────────────────────────────────────────────────┐
│                    API Layer                        │
│                  (routers/*.py)                     │
│  - Definição de endpoints                           │
│  - Validação de entrada                             │
│  - Respostas HTTP                                   │
└─────────────────────┬───────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────────┐
│                  Schema Layer                       │
│                 (schemas/*.py)                      │
│  - Validação Pydantic                               │
│  - Serialização/Desserialização                     │
│  - Type hints                                       │
└─────────────────────┬───────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────────┐
│                   Model Layer                       │
│                  (models/*.py)                      │
│  - Definição de tabelas                             │
│  - Relacionamentos                                  │
│  - ORM SQLAlchemy                                   │
└─────────────────────┬───────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────────┐
│                  Database Layer                     │
│                (core/database.py)                   │
│  - Conexão com banco                                │
│  - Sessões assíncronas                              │
│  - Engine configuration                             │
└─────────────────────────────────────────────────────┘
```

## 🔄 Fluxo de Requisição

```
Cliente → Router → Schema (validação) → Security (auth) → Model → Database
                                                              ↓
Cliente ← Response ← Schema (serialização) ← Controller ← Result
```

## 📝 Convenções de Nomes

### Diretórios

- **minúsculas** com **snake_case** para múltiplas palavras
- Ex: `car_api`, `core`, `schemas`

### Arquivos Python

- **minúsculas** com **snake_case**
- Ex: `users.py`, `get_session.py`, `user_service.py`

### Classes

- **PascalCase**
- Ex: `User`, `CarSchema`, `BaseModel`

### Funções e Variáveis

- **snake_case**
- Ex: `get_current_user`, `user_id`, `create_access_token`

### Constantes

- **UPPER_SNAKE_CASE**
- Ex: `JWT_SECRET_KEY`, `DATABASE_URL`

## 📦 Dependências do Projeto

### Principais

| Dependência | Versão | Finalidade |
|-------------|--------|------------|
| fastapi | 0.133.x | Framework web |
| pydantic | 2.12.x | Validação de dados |
| sqlalchemy | 2.0.x | ORM |
| aiosqlite | 0.22.x | Driver SQLite async |
| pydantic-settings | 2.13.x | Configurações |
| alembic | 1.18.x | Migrações |
| pwdlib[argon2] | 0.3.x | Hash de senha |
| pyjwt | 2.11.x | JWT tokens |

### Desenvolvimento

| Dependência | Versão | Finalidade |
|-------------|--------|------------|
| ruff | 0.15.x | Linting/Formatting |
| taskipy | 1.14.x | Tasks/Scripts |
| mkdocs | 1.6.x | Documentação |
| mkdocs-material | 9.7.x | Tema da documentação |

## 🚀 Próximo Passo

Com a estrutura compreendida, prossiga para [API Endpoints](api-endpoints.md).

---

**Dúvidas?** Consulte [Modelagem do Sistema](system-modeling.md) para entender os modelos de dados.
