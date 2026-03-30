# Desenvolvimento

Este guia fornece informações detalhadas para desenvolvedores que trabalham na Fastcar API.

## 🚀 Configuração do Ambiente de Desenvolvimento

### 1. Clonar o Repositório

```bash
git clone <url-do-repositorio>
cd car_api
```

### 2. Instalar Dependências

```bash
# Instalar Poetry (se não tiver) - recomendado via pipx
pipx install poetry

# Ou via método oficial
# curl -sSL https://install.python-poetry.org | python3 -

# Instalar dependências do projeto
poetry install --with dev
```

### 3. Configurar Variáveis de Ambiente

```bash
# Copiar arquivo .env de exemplo (se necessário)
cp .env.example .env

# Editar .env com suas configurações
# Ver seção de Configuração para detalhes
```

### 4. Executar Migrações

```bash
# Ativar ambiente virtual
poetry shell

# Executar migrações
alembic upgrade head
```

### 5. Rodar em Modo Desenvolvimento

```bash
# Usando taskipy
poetry run task run

# Ou diretamente
poetry run fastapi dev car_api/app.py
```

## 🛠️ Ferramentas de Desenvolvimento

### Ruff (Linting e Formatação)

```bash
# Verificar código
poetry run task lint

# Formatar código
poetry run task format

# Corrigir automaticamente
poetry run ruff check --fix
```

### MkDocs (Documentação)

```bash
# Servir documentação localmente
poetry run task docs

# Acessar em http://127.0.0.1:8001
```

### Alembic (Migrações)

```bash
# Criar nova migração
alembic revision --autogenerate -m "Descrição da migração"

# Aplicar migrações
alembic upgrade head

# Reverter migração
alembic downgrade -1

# Ver histórico
alembic history
```

## 📝 Workflow de Desenvolvimento

### 1. Criar Branch para Feature

```bash
git checkout -b feature/nova-feature
# ou
git checkout -b fix/correcao-bug
# ou
git checkout -b docs/atualizacao-docs
```

### 2. Desenvolver a Feature

```bash
# Fazer alterações no código
# Seguir guidelines em guidelines.md

# Verificar código
poetry run task lint

# Formatar código
poetry run task format
```

### 3. Testar Alterações

```bash
# Rodar testes
poetry run pytest tests/

# Testar manualmente via Swagger
# http://localhost:8000/docs
```

### 4. Commit e Push

```bash
# Adicionar alterações
git add .

# Commit com mensagem descritiva
git commit -m "feat: adicionar nova funcionalidade X"

# Push para branch
git push origin feature/nova-feature
```

### 5. Criar Pull Request

- Abrir PR no GitHub/GitLab
- Descrever mudanças
- Aguardar code review
- Aprovar e mergear

## 🏗️ Adicionando Novos Endpoints

### 1. Criar Router

```python
# car_api/routers/exemplo.py
from fastapi import APIRouter, Depends, HTTPException, status
from sqlalchemy.ext.asyncio import AsyncSession

from car_api.core.database import get_session

router = APIRouter()


@router.get('/exemplo')
async def get_exemplo(db: AsyncSession = Depends(get_session)):
    return {'message': 'Exemplo'}
```

### 2. Criar Schemas

```python
# car_api/schemas/exemplo.py
from pydantic import BaseModel


class ExemploSchema(BaseModel):
    name: str


class ExemploPublicSchema(BaseModel):
    id: int
    name: str
```

### 3. Criar Model

```python
# car_api/models/exemplo.py
from datetime import datetime
from sqlalchemy.orm import Mapped, mapped_column

from car_api.models import Base


class Exemplo(Base):
    __tablename__ = 'exemplos'
    
    id: Mapped[int] = mapped_column(primary_key=True)
    name: Mapped[str]
    created_at: Mapped[datetime] = mapped_column(server_default=func.now())
```

### 4. Registrar Router no App

```python
# car_api/app.py
from car_api.routers import exemplo

app.include_router(
    router=exemplo.router,
    prefix='/api/v1/exemplo',
    tags=['exemplo'],
)
```

### 5. Criar Migração

```bash
alembic revision --autogenerate -m "create exemplos table"
alembic upgrade head
```

## 🔧 Debugging

### Logs no FastAPI

```python
import logging

logger = logging.getLogger(__name__)

@router.get('/exemplo')
async def get_exemplo():
    logger.info("Buscando exemplo")
    try:
        # código
        logger.debug("Dados encontrados")
    except Exception as e:
        logger.error(f"Erro ao buscar exemplo: {e}")
        raise
```

### Debug com breakpoint

```bash
# Instalar pdb++
poetry add --group dev pdbpp

# Usar no código
import pdb; pdb.set_trace()
```

### Debug no VS Code

```json
// .vscode/launch.json
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "FastAPI",
      "type": "debugpy",
      "request": "launch",
      "module": "fastapi",
      "console": "integratedTerminal",
      "args": ["dev", "car_api/app.py"]
    }
  ]
}
```

## 📊 Trabalhando com Banco de Dados

### Query Assíncrona

```python
from sqlalchemy import select
from sqlalchemy.ext.asyncio import AsyncSession

async def get_users(db: AsyncSession):
    result = await db.execute(select(User))
    return result.scalars().all()
```

### Transações

```python
async def create_user_with_cars(db: AsyncSession):
    async with db.begin():
        user = User(username='test', email='test@test.com')
        db.add(user)
        await db.flush()  # Para obter o ID
        
        car = Car(model='Test', owner_id=user.id, ...)
        db.add(car)
        
        # Commit automático no final do bloco
```

### Relacionamentos

```python
# Carregar relacionamentos
from sqlalchemy.orm import selectinload

result = await db.execute(
    select(Car)
    .options(selectinload(Car.brand), selectinload(Car.owner))
    .where(Car.id == car_id)
)
car = result.scalar_one()

# Acessar relacionamentos
print(car.brand.name)
print(car.owner.username)
```

## 🧪 Escrevendo Testes

### Estrutura de Teste

```python
# tests/test_users.py
import pytest
from fastapi.testclient import TestClient
from car_api.app import app

client = TestClient(app)


def test_create_user():
    response = client.post(
        '/api/v1/users/',
        json={
            'username': 'testuser',
            'email': 'test@test.com',
            'password': '123456'
        }
    )
    assert response.status_code == 201
    assert response.json()['username'] == 'testuser'
```

### Fixtures

```python
# tests/conftest.py
import pytest
from sqlalchemy.ext.asyncio import create_async_engine, AsyncSession
from sqlalchemy.orm import sessionmaker

@pytest.fixture
async def db_session():
    engine = create_async_engine('sqlite+aiosqlite:///test.db')
    async_session = sessionmaker(engine, class_=AsyncSession)
    async with async_session() as session:
        yield session
```

## 📦 Gerenciando Dependências

### Adicionar Dependência

```bash
# Dependência principal
poetry add package-name

# Dependência de desenvolvimento
poetry add --group dev package-name

# Dependência opcional
poetry add --optional package-name
```

### Atualizar Dependências

```bash
# Atualizar todas
poetry update

# Atualizar específica
poetry update package-name
```

### Remover Dependência

```bash
poetry remove package-name
```

## 🚨 Solução de Problemas Comuns

### Erro: "No module named 'car_api'"

```bash
# Certificar-se de estar no ambiente virtual
poetry shell

# Ou usar poetry run
poetry run python -m car_api.app
```

### Erro: "Table already exists"

```bash
# Resetar banco de dados
rm car.db
alembic upgrade head
```

### Erro: "Token has expired"

```bash
# Aumentar expiração no .env
JWT_EXPIRATION_MINUTES=60

# Ou usar refresh token
POST /api/v1/auth/refresh_token
```

### Erro: "Ruff check failed"

```bash
# Verificar erros específicos
poetry run ruff check

# Corrigir automaticamente
poetry run ruff check --fix

# Formatar
poetry run ruff format
```

## 📈 Performance e Otimização

### Query Optimization

```python
# ✅ Usar selectinload para evitar N+1
result = await db.execute(
    select(Car).options(
        selectinload(Car.brand),
        selectinload(Car.owner)
    )
)

# ❌ Evitar N+1 queries
cars = await db.execute(select(Car))
for car in cars.scalars():
    print(car.brand.name)  # Query adicional para cada carro
```

### Pagination

```python
# Sempre usar paginação em listas
@router.get('/')
async def list_items(
    offset: int = Query(0, ge=0),
    limit: int = Query(100, ge=1, le=100),
):
    query = select(Item).offset(offset).limit(limit)
```

### Cache

```python
from functools import lru_cache

@lru_cache(maxsize=128)
def get_settings():
    return Settings()
```

## 🔒 Segurança no Desenvolvimento

### Nunca commitar:

- [ ] Arquivo `.env`
- [ ] Chaves de API
- [ ] Senhas
- [ ] Tokens de acesso

### Verificar antes do commit:

```bash
# Verificar se .env está no .gitignore
cat .gitignore | grep .env

# Verificar arquivos staged
git status

# Verificar mudanças
git diff --cached
```

## 📚 Recursos Adicionais

### Documentação Oficial

- [FastAPI](https://fastapi.tiangolo.com/)
- [SQLAlchemy](https://docs.sqlalchemy.org/)
- [Pydantic](https://docs.pydantic.dev/)
- [Alembic](https://alembic.sqlalchemy.org/)

### Ferramentas

- [Ruff](https://docs.astral.sh/ruff/)
- [Poetry](https://python-poetry.org/docs/)
- [MkDocs](https://www.mkdocs.org/)

## 🚀 Próximo Passo

Com o ambiente configurado, prossiga para [Testes](testing.md).

---

**Dúvidas?** Consulte [API Endpoints](api-endpoints.md) ou [Guidelines](guidelines.md).
