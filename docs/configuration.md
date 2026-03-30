# Configuração do Projeto

Este documento descreve todas as opções de configuração disponíveis para a Fastcar API.

## 📁 Arquivo de Configuração

As configurações são gerenciadas através do arquivo `.env` na raiz do projeto e da classe `Settings` em `car_api/core/settings.py`.

## 🔐 Variáveis de Ambiente

### Configurações Principais

| Variável | Tipo | Obrigatória | Valor Padrão | Descrição |
|----------|------|-------------|--------------|-----------|
| `DATABASE_URL` | string | ✅ Sim | - | URL de conexão com o banco de dados |
| `JWT_SECRET_KEY` | string | ✅ Sim | - | Chave secreta para assinatura de tokens JWT |
| `JWT_ALGORITHM` | string | ❌ Não | `HS256` | Algoritmo de criptografia do JWT |
| `JWT_EXPIRATION_MINUTES` | int | ❌ Não | `30` | Tempo de expiração do token em minutos |

### Exemplo de Arquivo `.env`

```bash
# .env

# Configuração do Banco de Dados
DATABASE_URL='sqlite+aiosqlite:///car.db'

# Configuração JWT
JWT_SECRET_KEY='Ge8gafuKvuioXhtLoTwuv7g4xOZ29lZsWShFA1BgtDIbdDZURp6Sw8cvE9iKmehK'
JWT_ALGORITHM='HS256'
JWT_EXPIRATION_MINUTES=30
```

## 🗄️ Configuração do Banco de Dados

### SQLite (Desenvolvimento)

```bash
DATABASE_URL='sqlite+aiosqlite:///car.db'
```

### PostgreSQL (Produção)

```bash
DATABASE_URL='postgresql+asyncpg://usuario:senha@localhost:5432/nome_do_banco'
```

### MySQL (Produção)

```bash
DATABASE_URL='mysql+aiomysql://usuario:senha@localhost:3306/nome_do_banco'
```

### Drivers Necessários

Para usar bancos diferentes do SQLite, instale os drivers apropriados:

```bash
# PostgreSQL
poetry add asyncpg

# MySQL
poetry add aiomysql
```

## 🔑 Configuração JWT

### Gerar Chave Secreta

```bash
# Python
python -c "import secrets; print(secrets.token_urlsafe(64))"

# OpenSSL
openssl rand -hex 64
```

### Melhores Práticas

- ✅ Use chaves com pelo menos 64 caracteres
- ✅ Armazene a chave em variáveis de ambiente seguras
- ✅ Nunca commit a chave no versionamento
- ✅ Rotacione a chave periodicamente em produção

### Configurações Avançadas

```bash
# Tempo de expiração estendido (ex: 8 horas)
JWT_EXPIRATION_MINUTES=480

# Algoritmo alternativo
JWT_ALGORITHM='HS512'
```

## 🛠️ Classe Settings

A classe `Settings` em `car_api/core/settings.py` gerencia todas as configurações:

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

### Uso no Código

```python
from car_api.core.settings import Settings

settings = Settings()
print(settings.DATABASE_URL)
print(settings.JWT_SECRET_KEY)
```

## 📊 Configurações do Ruff

O arquivo `pyproject.toml` contém configurações de linting e formatação:

```toml
[tool.ruff]
line-length = 79
exclude = [
    ".git",
    "__pycache__",
    "alembic",
    "migrations",
]

[tool.ruff.lint]
preview = true
select = ['I', 'F', 'E', 'W', 'PL', 'PT']
ignore = ['PLR2004', 'PLR0917', 'PLR0913']

[tool.ruff.format]
preview = true
quote-style = 'single'
```

### Opções Disponíveis

| Configuração | Descrição |
|--------------|-----------|
| `line-length` | Comprimento máximo da linha (padrão: 79) |
| `quote-style` | Estilo de aspas ('single' ou 'double') |
| `select` | Regras de linting habilitadas |
| `ignore` | Regras de linting ignoradas |

## 📝 Configurações do Taskipy

Tasks disponíveis no `pyproject.toml`:

```toml
[tool.taskipy.tasks]
lint = 'ruff check'
pre_format = 'ruff check --fix'
format = 'ruff format'
run = 'fastapi dev car_api/app.py'
docs = 'mkdocs serve -a 127.0.0.1:8001'
```

### Executando Tasks

```bash
# Rodar lint
poetry run task lint

# Formatar código
poetry run task format

# Rodar aplicação
poetry run task run

# Rodar documentação
poetry run task docs
```

## 🌍 Variáveis de Ambiente por Ambiente

### Desenvolvimento

```bash
DATABASE_URL='sqlite+aiosqlite:///car_dev.db'
JWT_SECRET_KEY='dev-secret-key-not-for-production'
JWT_EXPIRATION_MINUTES=60
```

### Testes

```bash
DATABASE_URL='sqlite+aiosqlite:///car_test.db'
JWT_SECRET_KEY='test-secret-key'
JWT_EXPIRATION_MINUTES=5
```

### Produção

```bash
DATABASE_URL='postgresql+asyncpg://user:pass@db:5432/car_prod'
JWT_SECRET_KEY='<chave-gerada-segura>'
JWT_EXPIRATION_MINUTES=30
```

## 🔒 Segurança

### Checklist de Segurança

- [ ] JWT_SECRET_KEY única e segura em produção
- [ ] Arquivo `.env` no `.gitignore`
- [ ] Banco de dados de produção com senha forte
- [ ] HTTPS habilitado em produção
- [ ] Logs não expõem informações sensíveis

### .gitignore

Certifique-se de que o `.env` está no `.gitignore`:

```gitignore
# Variáveis de ambiente
.env
.env.local
.env.*.local

# Banco de dados SQLite
*.db
*.sqlite
*.sqlite3

# Python
__pycache__/
*.py[cod]
*$py.class
*.so
```

## 🧪 Validação de Configuração

### Script de Validação

Crie um script para validar as configurações:

```python
# validate_config.py
from car_api.core.settings import Settings

try:
    settings = Settings()
    print("✅ Configurações válidas!")
    print(f"Database: {settings.DATABASE_URL[:30]}...")
    print(f"JWT Expiration: {settings.JWT_EXPIRATION_MINUTES} minutos")
except Exception as e:
    print(f"❌ Erro de configuração: {e}")
```

### Executar Validação

```bash
poetry run python validate_config.py
```

## 📋 Resumo

| Configuração | Desenvolvimento | Produção |
|--------------|-----------------|----------|
| Database | SQLite | PostgreSQL |
| JWT Expiration | 60 min | 30 min |
| Debug | Habilitado | Desabilitado |
| Logs | Detalhado | Resumido |

## 🚀 Próximo Passo

Com a configuração concluída, prossiga para [Guidelines e Padrões](guidelines.md).

---

**Dúvidas?** Consulte a seção de [API Endpoints](api-endpoints.md) para entender como usar a API configurada.
