# Autenticação e Segurança

Este documento detalha o sistema de autenticação e segurança implementado na Fastcar API.

## 🔐 Visão Geral

A API utiliza **JWT (JSON Web Tokens)** para autenticação stateless, combinado com **Argon2** para hash de senhas, proporcionando uma camada robusta de segurança.

## 📋 Componentes de Segurança

### 1. Hash de Senhas (Argon2)

O Argon2 é o algoritmo recomendado para hash de senhas, vencedor do Password Hashing Competition.

```python
from pwdlib import PasswordHash

pwd_context = PasswordHash.recommended()

def get_password_hash(password: str) -> str:
    return pwd_context.hash(password)

def verify_password(plain_password: str, hashed_password: str) -> bool:
    return pwd_context.verify(plain_password, hashed_password)
```

**Características:**

- ✅ Resistente a ataques de GPU
- ✅ Resistente a ataques de side-channel
- ✅ Salt automático e embutido no hash
- ✅ Configuração de custo ajustável

### 2. JWT (JSON Web Tokens)

Tokens JWT são usados para autenticação stateless.

```python
import jwt
from datetime import datetime, timedelta, timezone

def create_access_token(data: dict) -> str:
    to_encode = data.copy()
    expire = datetime.now(timezone.utc) + timedelta(minutes=30)
    to_encode.update({'exp': expire})
    
    encoded_jwt = jwt.encode(
        to_encode,
        settings.JWT_SECRET_KEY,
        algorithm=settings.JWT_ALGORITHM
    )
    return encoded_jwt
```

**Estrutura do Token:**

```
Header.Payload.Signature

Header: {"alg": "HS256", "typ": "JWT"}
Payload: {"sub": "1", "exp": 1704067200}
Signature: HMACSHA256(base64UrlEncode(header) + "." + base64UrlEncode(payload), secret)
```

### 3. Validação de Token

```python
def verify_token(token: str) -> dict:
    try:
        payload = jwt.decode(
            token,
            settings.JWT_SECRET_KEY,
            algorithms=[settings.JWT_ALGORITHM]
        )
        return payload
    except jwt.ExpiredSignatureError:
        raise HTTPException(
            status_code=401,
            detail='Token has expired',
            headers={'WWW-Authenticate': 'Bearer'}
        )
    except jwt.InvalidTokenError:
        raise HTTPException(
            status_code=401,
            detail='Could not validate credentials',
            headers={'WWW-Authenticate': 'Bearer'}
        )
```

## 🔑 Fluxo de Autenticação

### 1. Registro de Usuário

```mermaid
sequenceDiagram
    participant C as Cliente
    participant API as API
    participant DB as Database
    
    C->>API: POST /users/ (username, email, password)
    API->>API: Validar dados (Pydantic)
    API->>DB: Verificar username/email únicos
    DB-->>API: Disponível
    
    API->>API: Hash da senha (Argon2)
    API->>DB: INSERT user
    DB-->>API: User criado
    API-->>C: 201 Created (user sem senha)
```

**Exemplo de Request:**

```json
POST /api/v1/users/
Content-Type: application/json

{
  "username": "joaosilva",
  "email": "joao@email.com",
  "password": "senha123"
}
```

**Exemplo de Response:**

```json
{
  "id": 1,
  "username": "joaosilva",
  "email": "joao@email.com",
  "created_at": "2024-01-15T10:30:00",
  "updated_at": "2024-01-15T10:30:00"
}
```

### 2. Login (Obter Token)

```mermaid
sequenceDiagram
    participant C as Cliente
    participant API as API
    participant DB as Database
    participant JWT as JWT Module
    
    C->>API: POST /auth/token (email, password)
    API->>DB: SELECT user WHERE email = ?
    DB-->>API: User (com hash)
    
    API->>API: verify_password(senha, hash)
    
    alt Senha correta
        API->>JWT: create_access_token({sub: user_id})
        JWT-->>API: token
        API-->>C: 200 OK {access_token, token_type}
    else Senha incorreta
        API-->>C: 401 Unauthorized
    end
```

**Exemplo de Request:**

```json
POST /api/v1/auth/token
Content-Type: application/json

{
  "email": "joao@email.com",
  "password": "senha123"
}
```

**Exemplo de Response:**

```json
{
  "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxIiwiZXhwIjoxNzA0MDY3MjAwfQ.abc123",
  "token_type": "bearer"
}
```

### 3. Refresh Token

```mermaid
sequenceDiagram
    participant C as Cliente
    participant API as API
    participant JWT as JWT Module
    
    C->>API: POST /auth/refresh_token (Authorization: Bearer token)
    API->>JWT: verify_token(token)
    JWT-->>API: payload {sub: user_id}
    
    API->>JWT: create_access_token({sub: user_id})
    JWT-->>API: new_token
    API-->>C: 200 OK {access_token, token_type}
```

**Exemplo de Request:**

```http
POST /api/v1/auth/refresh_token
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

**Exemplo de Response:**

```json
{
  "access_token": "novo_token_aqui",
  "token_type": "bearer"
}
```

### 4. Acesso a Endpoint Protegido

```mermaid
sequenceDiagram
    participant C as Cliente
    participant API as API
    participant JWT as JWT Module
    participant DB as Database
    
    C->>API: GET /cars/ (Authorization: Bearer token)
    API->>JWT: verify_token(token)
    JWT-->>API: payload {sub: user_id}
    
    API->>DB: SELECT user WHERE id = user_id
    DB-->>API: User
    
    API->>DB: SELECT cars WHERE owner_id = user_id
    DB-->>API: Cars list
    API-->>C: 200 OK (cars)
```

**Exemplo de Request:**

```http
GET /api/v1/cars/
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

## 🛡️ Validação de Propriedade (Ownership)

A API implementa validação de ownership para garantir que apenas o proprietário de um recurso possa modificá-lo ou excluí-lo.

```python
def verify_car_ownership(user: User, car_owner_id: int) -> None:
    if user.id != car_owner_id:
        raise HTTPException(
            status_code=status.HTTP_403_FORBIDDEN,
            detail='Not enough permissions to access this car',
        )
```

**Uso nos Endpoints:**

```python
@router.get('/{car_id}')
async def get_car(
    car_id: int,
    current_user: User = Depends(get_current_user),
    db: AsyncSession = Depends(get_session),
):
    car = await db.get(Car, car_id)
    
    if not car:
        raise HTTPException(status_code=404, detail='Carro não encontrado')
    
    # Verificar se o usuário é o proprietário
    verify_car_ownership(current_user, car.owner_id)
    
    return car
```

## 🔒 Configurações de Segurança

### Variáveis de Ambiente

```bash
# .env

# Chave secreta para JWT (gerar uma única para produção)
JWT_SECRET_KEY='sua-chave-secreta-de-64-caracteres'

# Algoritmo de assinatura
JWT_ALGORITHM='HS256'

# Tempo de expiração em minutos
JWT_EXPIRATION_MINUTES=30
```

### Gerar Chave Secreta Segura

```bash
# Python
python -c "import secrets; print(secrets.token_urlsafe(64))"

# OpenSSL
openssl rand -hex 64
```

### Validações de Senha

```python
from pydantic import field_validator

class UserSchema(BaseModel):
    username: str
    email: EmailStr
    password: str
    
    @field_validator('username')
    def username_min_length(cls, v):
        if len(v) < 3:
            raise ValueError('Username deve conter pelo menos 3 caracteres')
        return v
    
    @field_validator('password')
    def password_min_length(cls, v):
        if len(v) < 6:
            raise ValueError('Senha deve conter pelo menos 6 caracteres')
        return v
```

## 🚨 Tratamento de Erros de Autenticação

### Códigos de Status

| Código | Descrição | Quando Ocorre |
|--------|-----------|---------------|
| `401 Unauthorized` | Token ausente, inválido ou expirado | Token mal formado, expirado ou usuário não encontrado |
| `403 Forbidden` | Token válido mas sem permissão | Usuário tenta acessar recurso de outro usuário |
| `400 Bad Request` | Dados de login inválidos | Email ou senha incorretos (mensagem genérica) |

### Mensagens de Erro

```python
# Token expirado
{
  "detail": "Token has expired"
}

# Token inválido
{
  "detail": "Could not validate credentials"
}

# Credenciais incorretas
{
  "detail": "Incorrect email or password"
}

# Sem permissão
{
  "detail": "Not enough permissions to access this car"
}
```

## 🔐 Melhores Práticas de Segurança

### 1. Armazenamento de Senhas

✅ **Correto:**
- Usar Argon2 ou bcrypt para hash
- Salt automático e único por senha
- Nunca armazenar senhas em texto plano

❌ **Incorreto:**
- Hash MD5 ou SHA1
- Salt fixo ou ausente
- Log de senhas

### 2. Tokens JWT

✅ **Correto:**
- Chave secreta forte (64+ caracteres)
- Expiração curta (30-60 minutos)
- Usar HTTPS em produção
- Implementar refresh token

❌ **Incorreto:**
- Chave fraca ou padrão
- Tokens sem expiração
- Transmitir token via URL
- Commit da chave no Git

### 3. Headers de Segurança

```python
# Adicionar headers de segurança
@app.middleware("http")
async def add_security_headers(request: Request, call_next):
    response = await call_next(request)
    response.headers["X-Content-Type-Options"] = "nosniff"
    response.headers["X-Frame-Options"] = "DENY"
    response.headers["X-XSS-Protection"] = "1; mode=block"
    return response
```

### 4. Rate Limiting

```python
from slowapi import Limiter
from slowapi.util import get_remote_address

limiter = Limiter(key_func=get_remote_address)

@app.post("/auth/token")
@limiter.limit("5/minute")
async def token(request: Request, login_data: LoginRequest):
    # Máximo 5 tentativas por minuto
    pass
```

## 🔍 Decodificando um Token JWT

### Exemplo de Token

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxIiwiZXhwIjoxNzA0MDY3MjAwfQ.abc123
```

### Decodificar (sem verificar assinatura)

```python
import jwt

token = "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."

# Decodificar sem verificar (apenas para debug)
payload = jwt.decode(token, options={"verify_signature": False})
print(payload)
# {'sub': '1', 'exp': 1704067200}
```

### Decodificar (verificando assinatura)

```python
from car_api.core.settings import Settings

settings = Settings()

try:
    payload = jwt.decode(
        token,
        settings.JWT_SECRET_KEY,
        algorithms=[settings.JWT_ALGORITHM]
    )
    user_id = payload.get('sub')
    print(f"User ID: {user_id}")
except jwt.ExpiredSignatureError:
    print("Token expirado")
except jwt.InvalidTokenError:
    print("Token inválido")
```

## 📊 Comparação de Métodos de Autenticação

| Método | Vantagens | Desvantagens | Uso na API |
|--------|-----------|--------------|------------|
| **JWT** | Stateless, escalável, sem sessão | Token não pode ser revogado facilmente | ✅ Usado |
| **Session** | Revogação fácil, controle total | Requer armazenamento de sessão | ❌ Não usado |
| **OAuth2** | Terceirização de auth, SSO | Complexidade adicional | ❌ Não usado |
| **API Key** | Simples, bom para servidores | Menos seguro para usuários | ❌ Não usado |

## 🔒 Checklist de Segurança

### Desenvolvimento

- [ ] Senhas com hash Argon2
- [ ] JWT com expiração configurada
- [ ] Validação de ownership implementada
- [ ] Input validation com Pydantic
- [ ] HTTPS em produção
- [ ] Chave JWT secreta e única
- [ ] `.env` no `.gitignore`

### Produção

- [ ] HTTPS/TLS habilitado
- [ ] Rate limiting configurado
- [ ] Logs não expõem dados sensíveis
- [ ] Headers de segurança configurados
- [ ] CORS configurado corretamente
- [ ] Banco de dados com senha forte
- [ ] Backup de chaves seguro

## 🚀 Próximo Passo

Com a segurança compreendida, prossiga para [Desenvolvimento](development.md).

---

**Dúvidas?** Consulte [Configuração](configuration.md) para detalhes sobre variáveis de ambiente de segurança.
