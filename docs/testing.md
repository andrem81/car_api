# Testes

Este documento descreve a estratégia de testes e como escrever e executar testes na Fastcar API.

## 📋 Visão Geral

A Fastcar API utiliza testes automatizados para garantir a qualidade e confiabilidade do código. A estrutura de testes inclui:

- **Testes Unitários**: Testam funções e métodos individuais
- **Testes de Integração**: Testam a integração entre componentes
- **Testes de API**: Testam endpoints HTTP

## 🛠️ Configuração

### Dependências de Teste

```toml
# pyproject.toml
[tool.poetry.group.dev.dependencies]
pytest = "^7.0.0"
pytest-asyncio = "^0.21.0"
httpx = "^0.24.0"
```

### Estrutura de Diretórios

```
tests/
├── __init__.py
├── conftest.py          # Fixtures e configurações
├── test_auth.py         # Testes de autenticação
├── test_users.py        # Testes de usuários
├── test_cars.py         # Testes de carros
└── test_brands.py       # Testes de marcas
```

## 🏃 Executando Testes

### Comandos Básicos

```bash
# Rodar todos os testes
poetry run pytest tests/

# Rodar com verbose
poetry run pytest tests/ -v

# Rodar teste específico
poetry run pytest tests/test_users.py -v

# Rodar com coverage
poetry run pytest tests/ --cov=car_api --cov-report=html

# Rodar testes marcados
poetry run pytest tests/ -m "not slow"
```

### Tasks do Poetry

```bash
# Adicionar task no pyproject.toml
[tool.taskipy.tasks]
test = 'pytest tests/'
test_cov = 'pytest tests/ --cov=car_api --cov-report=html'
```

## 🔧 Fixtures

### conftest.py

```python
# tests/conftest.py
import pytest
import asyncio
from typing import AsyncGenerator, Generator
from sqlalchemy.ext.asyncio import AsyncSession, create_async_engine, async_sessionmaker
from fastapi.testclient import TestClient

from car_api.app import app
from car_api.core.database import get_session
from car_api.models.base import Base


# Engine de teste
TEST_DATABASE_URL = "sqlite+aiosqlite:///test.db"


@pytest.fixture(scope="session")
def event_loop() -> Generator:
    """Criar event loop para testes async."""
    loop = asyncio.get_event_loop_policy().new_event_loop()
    yield loop
    loop.close()


@pytest.fixture(scope="function")
async def db_engine():
    """Criar engine de banco de dados de teste."""
    engine = create_async_engine(TEST_DATABASE_URL, echo=True)
    
    async with engine.begin() as conn:
        await conn.run_sync(Base.metadata.create_all)
    
    yield engine
    
    async with engine.begin() as conn:
        await conn.run_sync(Base.metadata.drop_all)
    
    await engine.dispose()


@pytest.fixture
async def db_session(db_engine) -> AsyncGenerator[AsyncSession, None]:
    """Criar sessão de banco de dados de teste."""
    async_session = async_sessionmaker(
        db_engine, class_=AsyncSession, expire_on_commit=False
    )
    
    async with async_session() as session:
        yield session


@pytest.fixture
def client(db_session) -> Generator[TestClient, None, None]:
    """Criar cliente de teste para a API."""
    
    async def override_get_session():
        yield db_session
    
    app.dependency_overrides[get_session] = override_get_session
    
    with TestClient(app) as test_client:
        yield test_client
    
    app.dependency_overrides.clear()


@pytest.fixture
async def test_user(db_session):
    """Criar usuário de teste."""
    from car_api.models.users import User
    from car_api.core.security import get_password_hash
    
    user = User(
        username='testuser',
        email='test@test.com',
        password=get_password_hash('test123')
    )
    
    db_session.add(user)
    await db_session.commit()
    await db_session.refresh(user)
    
    return user


@pytest.fixture
async def auth_token(client: TestClient, test_user) -> str:
    """Obter token de autenticação para teste."""
    response = client.post(
        '/api/v1/auth/token',
        json={'email': 'test@test.com', 'password': 'test123'}
    )
    return response.json()['access_token']
```

## 📝 Escrevendo Testes

### Testes de Autenticação

```python
# tests/test_auth.py
import pytest
from fastapi.testclient import TestClient


class TestAuthToken:
    """Testes para autenticação e tokens."""
    
    def test_login_success(self, client: TestClient, test_user):
        """Testar login bem-sucedido."""
        response = client.post(
            '/api/v1/auth/token',
            json={'email': 'test@test.com', 'password': 'test123'}
        )
        
        assert response.status_code == 200
        data = response.json()
        assert 'access_token' in data
        assert data['token_type'] == 'bearer'
    
    def test_login_invalid_email(self, client: TestClient):
        """Testar login com email inválido."""
        response = client.post(
            '/api/v1/auth/token',
            json={'email': 'invalid@test.com', 'password': 'test123'}
        )
        
        assert response.status_code == 401
        assert response.json()['detail'] == 'Incorrect email or password'
    
    def test_login_invalid_password(self, client: TestClient, test_user):
        """Testar login com senha inválida."""
        response = client.post(
            '/api/v1/auth/token',
            json={'email': 'test@test.com', 'password': 'wrongpassword'}
        )
        
        assert response.status_code == 401
    
    def test_refresh_token(self, client: TestClient, auth_token: str):
        """Testar refresh de token."""
        response = client.post(
            '/api/v1/auth/refresh_token',
            headers={'Authorization': f'Bearer {auth_token}'}
        )
        
        assert response.status_code == 200
        assert 'access_token' in response.json()
```

### Testes de Usuários

```python
# tests/test_users.py
import pytest
from fastapi.testclient import TestClient


class TestUsers:
    """Testes para endpoints de usuários."""
    
    def test_create_user_success(self, client: TestClient):
        """Testar criação de usuário com sucesso."""
        user_data = {
            'username': 'newuser',
            'email': 'newuser@test.com',
            'password': 'password123'
        }
        
        response = client.post('/api/v1/users/', json=user_data)
        
        assert response.status_code == 201
        data = response.json()
        assert data['username'] == 'newuser'
        assert data['email'] == 'newuser@test.com'
        assert 'id' in data
        assert 'password' not in data
    
    def test_create_user_duplicate_username(self, client: TestClient, test_user):
        """Testar criação de usuário com username duplicado."""
        user_data = {
            'username': 'testuser',  # Mesmo username do test_user
            'email': 'different@test.com',
            'password': 'password123'
        }
        
        response = client.post('/api/v1/users/', json=user_data)
        
        assert response.status_code == 400
        assert 'Username já está em uso' in response.json()['detail']
    
    def test_create_user_duplicate_email(self, client: TestClient, test_user):
        """Testar criação de usuário com email duplicado."""
        user_data = {
            'username': 'differentuser',
            'email': 'test@test.com',  # Mesmo email do test_user
            'password': 'password123'
        }
        
        response = client.post('/api/v1/users/', json=user_data)
        
        assert response.status_code == 400
        assert 'Email já está em uso' in response.json()['detail']
    
    def test_create_user_invalid_email(self, client: TestClient):
        """Testar criação de usuário com email inválido."""
        user_data = {
            'username': 'user',
            'email': 'invalid-email',
            'password': 'password123'
        }
        
        response = client.post('/api/v1/users/', json=user_data)
        
        assert response.status_code == 422  # Validation error
    
    def test_create_user_short_password(self, client: TestClient):
        """Testar criação de usuário com senha curta."""
        user_data = {
            'username': 'user',
            'email': 'user@test.com',
            'password': '12345'  # Menos de 6 caracteres
        }
        
        response = client.post('/api/v1/users/', json=user_data)
        
        assert response.status_code == 422
    
    def test_list_users(self, client: TestClient, test_user):
        """Testar listagem de usuários."""
        response = client.get('/api/v1/users/')
        
        assert response.status_code == 200
        data = response.json()
        assert 'users' in data
        assert len(data['users']) >= 1
    
    def test_get_user_by_id(self, client: TestClient, test_user):
        """Testar busca de usuário por ID."""
        response = client.get(f'/api/v1/users/{test_user.id}')
        
        assert response.status_code == 200
        data = response.json()
        assert data['id'] == test_user.id
        assert data['username'] == 'testuser'
    
    def test_get_user_not_found(self, client: TestClient):
        """Testar busca de usuário inexistente."""
        response = client.get('/api/v1/users/99999')
        
        assert response.status_code == 404
    
    def test_update_user(self, client: TestClient, test_user, auth_token: str):
        """Testar atualização de usuário."""
        update_data = {'username': 'updateduser'}
        
        response = client.put(
            f'/api/v1/users/{test_user.id}',
            json=update_data,
            headers={'Authorization': f'Bearer {auth_token}'}
        )
        
        assert response.status_code == 200
        assert response.json()['username'] == 'updateduser'
    
    def test_delete_user(self, client: TestClient, test_user, auth_token: str):
        """Testar exclusão de usuário."""
        response = client.delete(
            f'/api/v1/users/{test_user.id}',
            headers={'Authorization': f'Bearer {auth_token}'}
        )
        
        assert response.status_code == 204
        
        # Verificar que foi deletado
        response = client.get(f'/api/v1/users/{test_user.id}')
        assert response.status_code == 404
```

### Testes de Carros

```python
# tests/test_cars.py
import pytest
from fastapi.testclient import TestClient
from decimal import Decimal


class TestCars:
    """Testes para endpoints de carros."""
    
    @pytest.fixture
    async def test_brand(self, db_session):
        """Criar marca de teste."""
        from car_api.models.cars import Brand
        
        brand = Brand(name='TestBrand', description='Test Description')
        db_session.add(brand)
        await db_session.commit()
        await db_session.refresh(brand)
        return brand
    
    def test_create_car_success(
        self, client: TestClient, auth_token: str, test_brand, test_user
    ):
        """Testar criação de carro com sucesso."""
        car_data = {
            'model': 'Test Model',
            'factory_year': 2024,
            'model_year': 2024,
            'color': 'Red',
            'plate': 'ABC1D23',
            'fuel_type': 'flex',
            'transmission': 'automatic',
            'price': 99900.00,
            'description': 'Test car',
            'is_available': True,
            'brand_id': test_brand.id,
            'owner_id': test_user.id
        }
        
        response = client.post(
            '/api/v1/cars/',
            json=car_data,
            headers={'Authorization': f'Bearer {auth_token}'}
        )
        
        assert response.status_code == 201
        data = response.json()
        assert data['model'] == 'Test Model'
        assert data['plate'] == 'ABC1D23'
        assert 'brand' in data
        assert 'owner' in data
    
    def test_create_car_duplicate_plate(
        self, client: TestClient, auth_token: str, test_brand, test_user
    ):
        """Testar criação de carro com placa duplicada."""
        car_data = {
            'model': 'Test Model',
            'factory_year': 2024,
            'model_year': 2024,
            'color': 'Red',
            'plate': 'ABC1D23',
            'fuel_type': 'flex',
            'transmission': 'automatic',
            'price': 99900.00,
            'brand_id': test_brand.id,
            'owner_id': test_user.id
        }
        
        # Criar primeiro carro
        client.post('/api/v1/cars/', json=car_data, headers={'Authorization': f'Bearer {auth_token}'})
        
        # Tentar criar segundo carro com mesma placa
        response = client.post(
            '/api/v1/cars/',
            json=car_data,
            headers={'Authorization': f'Bearer {auth_token}'}
        )
        
        assert response.status_code == 400
        assert 'Placa já está em uso' in response.json()['detail']
    
    def test_list_cars_with_filters(
        self, client: TestClient, auth_token: str, test_brand, test_user
    ):
        """Testar listagem de carros com filtros."""
        # Criar carros de teste
        car_data = {
            'model': 'Test Model',
            'factory_year': 2024,
            'model_year': 2024,
            'color': 'Red',
            'plate': 'ABC1D23',
            'fuel_type': 'flex',
            'transmission': 'automatic',
            'price': 99900.00,
            'brand_id': test_brand.id,
            'owner_id': test_user.id
        }
        client.post('/api/v1/cars/', json=car_data, headers={'Authorization': f'Bearer {auth_token}'})
        
        # Listar com filtro
        response = client.get(
            '/api/v1/cars/?fuel_type=flex&brand_id=1',
            headers={'Authorization': f'Bearer {auth_token}'}
        )
        
        assert response.status_code == 200
        data = response.json()
        assert 'cars' in data
    
    def test_get_car_not_owner(
        self, client: TestClient, auth_token: str, test_brand, test_user
    ):
        """Testar acesso a carro de outro usuário."""
        # Criar carro
        car_data = {
            'model': 'Test Model',
            'factory_year': 2024,
            'model_year': 2024,
            'color': 'Red',
            'plate': 'ABC1D23',
            'fuel_type': 'flex',
            'transmission': 'automatic',
            'price': 99900.00,
            'brand_id': test_brand.id,
            'owner_id': test_user.id
        }
        create_response = client.post(
            '/api/v1/cars/',
            json=car_data,
            headers={'Authorization': f'Bearer {auth_token}'}
        )
        car_id = create_response.json()['id']
        
        # Criar outro usuário
        user2_data = {
            'username': 'user2',
            'email': 'user2@test.com',
            'password': 'password123'
        }
        client.post('/api/v1/users/', json=user2_data)
        
        # Login como usuário 2
        token2_response = client.post(
            '/api/v1/auth/token',
            json={'email': 'user2@test.com', 'password': 'password123'}
        )
        token2 = token2_response.json()['access_token']
        
        # Tentar acessar carro do usuário 1
        response = client.get(
            f'/api/v1/cars/{car_id}',
            headers={'Authorization': f'Bearer {token2}'}
        )
        
        assert response.status_code == 403
```

## 🏷️ Marcando Testes

```python
# tests/test_slow.py
import pytest


@pytest.mark.slow
def test_slow_operation():
    """Teste que demora muito."""
    pass


@pytest.mark.integration
def test_integration():
    """Teste de integração."""
    pass


@pytest.mark.unit
def test_unit():
    """Teste unitário."""
    pass
```

### Executar por Mark

```bash
# Rodar testes lentos
poetry run pytest tests/ -m slow

# Rodar testes que não são lentos
poetry run pytest tests/ -m "not slow"

# Rodar testes de integração
poetry run pytest tests/ -m integration
```

## 📊 Coverage

### Configurar Coverage

```ini
# .coveragerc
[run]
source = car_api
omit = 
    */tests/*
    */migrations/*
    */__init__.py

[report]
exclude_lines =
    pragma: no cover
    def __repr__
    raise AssertionError
    raise NotImplementedError
```

### Executar Coverage

```bash
# Gerar relatório
poetry run pytest tests/ --cov=car_api --cov-report=html

# Abrir relatório
open htmlcov/index.html

# Relatório em terminal
poetry run pytest tests/ --cov=car_api --cov-report=term-missing
```

## 🚨 Testes em CI/CD

### GitHub Actions

```yaml
# .github/workflows/tests.yml
name: Tests

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: '3.13'
      
      - name: Install Poetry
        run: pip install poetry
      
      - name: Install dependencies
        run: poetry install --with dev
      
      - name: Run tests
        run: poetry run pytest tests/ -v --cov=car_api
      
      - name: Upload coverage
        uses: codecov/codecov-action@v3
```

## 📋 Checklist de Testes

Antes de submeter código:

- [ ] Testes unitários para novas funções
- [ ] Testes de integração para novos endpoints
- [ ] Testes de erro/cenários negativos
- [ ] Coverage mínimo de 80%
- [ ] Todos os testes passam
- [ ] Fixtures reutilizáveis quando possível

## 🚀 Próximo Passo

Com os testes configurados, prossiga para [Deploy](deployment.md).

---

**Dúvidas?** Consulte [Desenvolvimento](development.md) para mais detalhes sobre o workflow.
