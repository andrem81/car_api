# Guidelines e Padrões

Este documento estabelece as diretrizes e padrões de código a serem seguidos no desenvolvimento da Fastcar API.

## 📐 Princípios de Desenvolvimento

### SOLID

O projeto segue os princípios SOLID:

| Princípio | Descrição | Exemplo no Projeto |
|-----------|-----------|-------------------|
| **S** - Single Responsibility | Cada classe/função tem uma única responsabilidade | Routers separados por domínio (auth, users, cars, brands) |
| **O** - Open/Closed | Entidades abertas para extensão, fechadas para modificação | Schemas Pydantic estendíveis |
| **L** - Liskov Substitution | Subclasses devem ser substituíveis por suas bases | Enums `FuelType` e `TransmissionType` |
| **I** - Interface Segregation | Interfaces específicas ao invés de genéricas | Schemas específicos para create/update/response |
| **D** - Dependency Inversion | Dependência de abstrações | Injeção de dependência com `Depends()` |

### DRY (Don't Repeat Yourself)

- Evite código duplicado
- Reutilize funções utilitárias
- Use herança de schemas quando apropriado

### KISS (Keep It Simple, Stupid)

- Mantenha o código simples e legível
- Evite complexidade desnecessária
- Prefira soluções diretas

## 📝 Padrões de Nomenclatura

### Arquivos e Diretórios

```
✅ Correto:
car_api/routers/users.py
car_api/models/cars.py
car_api/schemas/auth.py
car_api/core/security.py

❌ Incorreto:
car_api/Routers/Users.py
car_api/models/Cars.py
car_api/Schemas/Auth.py
```

### Variáveis e Funções

```python
# Snake case para variáveis e funções
✅ Correto:
user_id = 123
def get_current_user():
    pass
def create_access_token():
    pass

❌ Incorreto:
userId = 123
def getCurrentUser():
    pass
```

### Classes

```python
# Pascal case para classes
✅ Correto:
class User(Base):
    pass
class CarSchema(BaseModel):
    pass

❌ Incorreto:
class user(Base):
    pass
class Car_schema(BaseModel):
    pass
```

### Constantes

```python
# Upper snake case para constantes
✅ Correto:
JWT_SECRET_KEY = "secret"
DATABASE_URL = "sqlite:///car.db"

❌ Incorreto:
jwtSecretKey = "secret"
databaseUrl = "sqlite:///car.db"
```

## 🐍 Padrões Python

### Type Hints

Sempre utilize type hints:

```python
# ✅ Correto
from typing import Optional, List
from sqlalchemy.orm import Mapped, mapped_column

class User(Base):
    id: Mapped[int] = mapped_column(primary_key=True)
    username: Mapped[str]
    email: Mapped[str]
    cars: Mapped[List['Car']] = relationship()

async def get_user(user_id: int) -> Optional[User]:
    pass

# ❌ Incorreto
class User(Base):
    id = mapped_column(primary_key=True)
    username = Column(String)
    
async def get_user(user_id):
    pass
```

### Docstrings

Utilize docstrings para funções públicas:

```python
def create_access_token(data: dict) -> str:
    """
    Cria um token de acesso JWT.
    
    Args:
        data: Dicionário contendo os dados para codificar (ex: {'sub': user_id})
    
    Returns:
        str: Token JWT codificado
    
    Raises:
        None
    """
    pass
```

### Tratamento de Erros

```python
# ✅ Correto - Específico
try:
    user = await db.get(User, user_id)
except HTTPException as e:
    raise e
except Exception as e:
    logger.error(f"Erro ao buscar usuário: {e}")
    raise HTTPException(status_code=500, detail="Erro interno")

# ❌ Incorreto - Genérico demais
try:
    user = await db.get(User, user_id)
except:
    pass
```

## 🏗️ Padrões de Arquitetura

### Estrutura de Camadas

```
Routers (Endpoints)
    ↓
Schemas (Validação)
    ↓
Models (ORM)
    ↓
Database (Conexão)
```

### Routers

```python
# ✅ Padrão estabelecido
from fastapi import APIRouter

router = APIRouter()

@router.post(
    path='/',
    status_code=status.HTTP_201_CREATED,
    response_model=UserPublicSchema,
    summary='Criar novo usuário',
)
async def create_user(
    user: UserSchema,
    db: AsyncSession = Depends(get_session),
):
    pass
```

### Schemas

```python
# ✅ Separação por propósito
class UserSchema(BaseModel):
    """Schema para criação de usuário"""
    pass

class UserUpdateSchema(BaseModel):
    """Schema para atualização de usuário"""
    pass

class UserPublicSchema(BaseModel):
    """Schema para resposta pública"""
    pass
```

### Models

```python
# ✅ Padrão SQLAlchemy 2.0
from sqlalchemy.orm import Mapped, mapped_column, relationship

class User(Base):
    __tablename__ = 'users'
    
    id: Mapped[int] = mapped_column(primary_key=True)
    username: Mapped[str] = mapped_column(unique=True)
    
    cars: Mapped[List['Car']] = relationship(back_populates='owner')
```

## 🔒 Padrões de Segurança

### Senhas

```python
# ✅ Hash de senha antes de salvar
from car_api.core.security import get_password_hash

hashed_password = get_password_hash(plain_password)

# ✅ Verificar senha
from car_api.core.security import verify_password

is_valid = verify_password(plain_password, hashed_password)
```

### Autenticação

```python
# ✅ Proteger endpoints com autenticação
@router.get('/protected')
async def protected_route(
    current_user: User = Depends(get_current_user),
):
    pass
```

### Validação de Propriedade

```python
# ✅ Verificar ownership antes de operações sensíveis
from car_api.core.security import verify_car_ownership

verify_car_ownership(current_user, car.owner_id)
```

## 📏 Padrões de Código

### Line Length

- **Máximo**: 79 caracteres
- **Ferramenta**: Ruff

### Quotes

- **Padrão**: Aspas simples (`'`)
- **Exceção**: Docstrings usam aspas triplas duplas (`"""`)

```python
# ✅ Correto
name = 'John'
message = 'Hello, World!'

"""Docstring com aspas triplas duplas."""

# ❌ Incorreto
name = "John"
message = "Hello, World!"
```

### Imports

```python
# ✅ Ordem correta
# 1. Standard library
from datetime import datetime
from typing import List, Optional

# 2. Third party
from fastapi import APIRouter, Depends
from sqlalchemy.orm import Mapped, mapped_column

# 3. First party
from car_api.core.database import get_session
from car_api.models.users import User
from car_api.schemas.users import UserSchema
```

### Espaçamento

```python
# ✅ Correto
# 2 linhas em branco entre classes/funções de nível superior
class User(Base):
    pass


class Car(Base):
    pass


# 1 linha em branco entre métodos
class UserService:
    def create(self):
        pass
    
    def delete(self):
        pass
```

## 🧪 Padrões de Testes

### Nomenclatura

```python
# ✅ Padrão
def test_create_user_success():
    pass

def test_create_user_invalid_email():
    pass

def test_get_user_not_found():
    pass
```

### Estrutura (AAA Pattern)

```python
def test_create_user():
    # Arrange
    user_data = {'username': 'test', 'email': 'test@test.com', 'password': '123456'}
    
    # Act
    response = client.post('/users/', json=user_data)
    
    # Assert
    assert response.status_code == 201
    assert response.json()['username'] == 'test'
```

## 📋 Checklist de Code Review

Antes de submeter código:

- [ ] Type hints em todas as funções
- [ ] Docstrings em funções públicas
- [ ] Tratamento de erros adequado
- [ ] Segue padrões de nomenclatura
- [ ] Ruff check passa sem erros
- [ ] Ruff format aplicado
- [ ] Testes adicionados/atualizados
- [ ] Variáveis de ambiente não commitadas
- [ ] Logs não expõem dados sensíveis

## 🛠️ Ferramentas

### Ruff

```bash
# Verificar código
poetry run task lint

# Formatar código
poetry run task format

# Corrigir automaticamente
poetry run ruff check --fix
```

### Configuração Ruff

```toml
[tool.ruff]
line-length = 79
quote-style = 'single'

[tool.ruff.lint]
select = ['I', 'F', 'E', 'W', 'PL', 'PT']
```

## 📚 Referências

- [PEP 8 - Style Guide for Python Code](https://pep8.org/)
- [PEP 484 - Type Hints](https://peps.python.org/pep-0484/)
- [FastAPI Best Practices](https://fastapi.tiangolo.com/)
- [SQLAlchemy 2.0 Documentation](https://docs.sqlalchemy.org/)
- [Pydantic Documentation](https://docs.pydantic.dev/)

## 🚀 Próximo Passo

Com os guidelines definidos, prossiga para [Estrutura do Projeto](structure.md).

---

**Dúvidas?** Consulte a seção de [Desenvolvimento](development.md) para exemplos práticos.
