# API Endpoints

Documentação completa de todos os endpoints da Fastcar API.

## 📋 Visão Geral

| Método | Endpoint | Descrição | Autenticação |
|--------|----------|-----------|--------------|
| `GET` | `/health_check` | Verificação de saúde | ❌ Não |
| `POST` | `/api/v1/auth/token` | Gerar token de acesso | ❌ Não |
| `POST` | `/api/v1/auth/refresh_token` | Atualizar token | ✅ Sim |
| `POST` | `/api/v1/users/` | Criar usuário | ❌ Não |
| `GET` | `/api/v1/users/` | Listar usuários | ❌ Não |
| `GET` | `/api/v1/users/{user_id}` | Buscar usuário | ❌ Não |
| `PUT` | `/api/v1/users/{user_id}` | Atualizar usuário | ✅ Sim |
| `DELETE` | `/api/v1/users/{user_id}` | Deletar usuário | ✅ Sim |
| `POST` | `/api/v1/brands/` | Criar marca | ✅ Sim |
| `GET` | `/api/v1/brands/` | Listar marcas | ✅ Sim |
| `GET` | `/api/v1/brands/{brand_id}` | Buscar marca | ✅ Sim |
| `PUT` | `/api/v1/brands/{brand_id}` | Atualizar marca | ✅ Sim |
| `DELETE` | `/api/v1/brands/{brand_id}` | Deletar marca | ✅ Sim |
| `POST` | `/api/v1/cars/` | Criar carro | ✅ Sim |
| `GET` | `/api/v1/cars/` | Listar carros | ✅ Sim |
| `GET` | `/api/v1/cars/{car_id}` | Buscar carro | ✅ Sim |
| `PUT` | `/api/v1/cars/{car_id}` | Atualizar carro | ✅ Sim |
| `DELETE` | `/api/v1/cars/{car_id}` | Deletar carro | ✅ Sim |

## 🔐 Autenticação

### Gerar Token de Acesso

```http
POST /api/v1/auth/token
Content-Type: application/json
```

**Request Body:**

```json
{
  "email": "usuario@email.com",
  "password": "senha123"
}
```

**Response (200 OK):**

```json
{
  "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "token_type": "bearer"
}
```

**Response (401 Unauthorized):**

```json
{
  "detail": "Incorrect email or password"
}
```

### Atualizar Token

```http
POST /api/v1/auth/refresh_token
Authorization: Bearer <token>
```

**Response (200 OK):**

```json
{
  "access_token": "novo_token_aqui",
  "token_type": "bearer"
}
```

## 👥 Usuários

### Criar Usuário

```http
POST /api/v1/users/
Content-Type: application/json
```

**Request Body:**

```json
{
  "username": "joaosilva",
  "email": "joao@email.com",
  "password": "senha123"
}
```

**Response (201 Created):**

```json
{
  "id": 1,
  "username": "joaosilva",
  "email": "joao@email.com",
  "created_at": "2024-01-15T10:30:00",
  "updated_at": "2024-01-15T10:30:00"
}
```

**Validações:**

- `username`: Mínimo 3 caracteres, único
- `email`: Email válido, único
- `password`: Mínimo 6 caracteres

**Response (400 Bad Request):**

```json
{
  "detail": "Username já está em uso"
}
```

### Listar Usuários

```http
GET /api/v1/users/?offset=0&limit=10&search=joao
```

**Query Parameters:**

| Parâmetro | Tipo | Padrão | Descrição |
|-----------|------|--------|-----------|
| `offset` | int | 0 | Registros para pular |
| `limit` | int | 100 | Limite de registros (1-100) |
| `search` | string | null | Buscar por username ou email |

**Response (200 OK):**

```json
{
  "users": [
    {
      "id": 1,
      "username": "joaosilva",
      "email": "joao@email.com",
      "created_at": "2024-01-15T10:30:00",
      "updated_at": "2024-01-15T10:30:00"
    }
  ],
  "offset": 0,
  "limit": 10
}
```

### Buscar Usuário por ID

```http
GET /api/v1/users/{user_id}
```

**Response (200 OK):**

```json
{
  "id": 1,
  "username": "joaosilva",
  "email": "joao@email.com",
  "created_at": "2024-01-15T10:30:00",
  "updated_at": "2024-01-15T10:30:00"
}
```

**Response (404 Not Found):**

```json
{
  "detail": "Usuário não encontrado"
}
```

### Atualizar Usuário

```http
PUT /api/v1/users/{user_id}
Authorization: Bearer <token>
Content-Type: application/json
```

**Request Body (todos opcionais):**

```json
{
  "username": "novo_username",
  "email": "novo@email.com",
  "password": "novasenha123"
}
```

**Response (200 OK):**

```json
{
  "id": 1,
  "username": "novo_username",
  "email": "novo@email.com",
  "created_at": "2024-01-15T10:30:00",
  "updated_at": "2024-01-15T11:00:00"
}
```

### Deletar Usuário

```http
DELETE /api/v1/users/{user_id}
Authorization: Bearer <token>
```

**Response (204 No Content)**

## 🚗 Marcas

### Criar Marca

```http
POST /api/v1/brands/
Authorization: Bearer <token>
Content-Type: application/json
```

**Request Body:**

```json
{
  "name": "Toyota",
  "description": "Fabricante japonesa de veículos",
  "is_active": true
}
```

**Response (201 Created):**

```json
{
  "id": 1,
  "name": "Toyota",
  "description": "Fabricante japonesa de veículos",
  "is_active": true,
  "created_at": "2024-01-15T10:30:00",
  "updated_at": "2024-01-15T10:30:00"
}
```

**Validações:**

- `name`: Mínimo 2 caracteres, único
- `description`: Opcional
- `is_active`: Padrão `true`

### Listar Marcas

```http
GET /api/v1/brands/?offset=0&limit=10&search=toyota&is_active=true
Authorization: Bearer <token>
```

**Query Parameters:**

| Parâmetro | Tipo | Padrão | Descrição |
|-----------|------|--------|-----------|
| `offset` | int | 0 | Registros para pular |
| `limit` | int | 100 | Limite de registros (1-100) |
| `search` | string | null | Buscar por nome |
| `is_active` | bool | null | Filtrar por status |

**Response (200 OK):**

```json
{
  "brands": [
    {
      "id": 1,
      "name": "Toyota",
      "description": "Fabricante japonesa de veículos",
      "is_active": true,
      "created_at": "2024-01-15T10:30:00",
      "updated_at": "2024-01-15T10:30:00"
    }
  ],
  "offset": 0,
  "limit": 10
}
```

### Buscar Marca por ID

```http
GET /api/v1/brands/{brand_id}
Authorization: Bearer <token>
```

**Response (200 OK):**

```json
{
  "id": 1,
  "name": "Toyota",
  "description": "Fabricante japonesa de veículos",
  "is_active": true,
  "created_at": "2024-01-15T10:30:00",
  "updated_at": "2024-01-15T10:30:00"
}
```

### Atualizar Marca

```http
PUT /api/v1/brands/{brand_id}
Authorization: Bearer <token>
Content-Type: application/json
```

**Request Body (todos opcionais):**

```json
{
  "name": "Toyota Motor",
  "description": "Nova descrição",
  "is_active": false
}
```

**Response (200 OK):**

```json
{
  "id": 1,
  "name": "Toyota Motor",
  "description": "Nova descrição",
  "is_active": false,
  "created_at": "2024-01-15T10:30:00",
  "updated_at": "2024-01-15T11:00:00"
}
```

### Deletar Marca

```http
DELETE /api/v1/brands/{brand_id}
Authorization: Bearer <token>
```

**Response (204 No Content)**

**Response (400 Bad Request):**

```json
{
  "detail": "Não é possível deletar marca que possui carros associados"
}
```

## 🚙 Carros

### Criar Carro

```http
POST /api/v1/cars/
Authorization: Bearer <token>
Content-Type: application/json
```

**Request Body:**

```json
{
  "model": "Corolla XEi",
  "factory_year": 2024,
  "model_year": 2024,
  "color": "Prata",
  "plate": "ABC1D23",
  "fuel_type": "flex",
  "transmission": "automatic",
  "price": 149900.00,
  "description": "Carro em perfeito estado",
  "is_available": true,
  "brand_id": 1,
  "owner_id": 1
}
```

**Enums:**

**FuelType:**
- `gasoline`
- `ethanol`
- `flex`
- `diesel`
- `electric`
- `hybrid`

**TransmissionType:**
- `manual`
- `automatic`
- `semi_automatic`
- `cvt`

**Response (201 Created):**

```json
{
  "id": 1,
  "model": "Corolla XEi",
  "factory_year": 2024,
  "model_year": 2024,
  "color": "Prata",
  "plate": "ABC1D23",
  "fuel_type": "flex",
  "transmission": "automatic",
  "price": 149900.00,
  "description": "Carro em perfeito estado",
  "is_available": true,
  "brand_id": 1,
  "owner_id": 1,
  "created_at": "2024-01-15T10:30:00",
  "updated_at": "2024-01-15T10:30:00",
  "brand": {
    "id": 1,
    "name": "Toyota",
    "description": "Fabricante japonesa de veículos",
    "is_active": true,
    "created_at": "2024-01-15T10:30:00",
    "updated_at": "2024-01-15T10:30:00"
  },
  "owner": {
    "id": 1,
    "username": "joaosilva",
    "email": "joao@email.com",
    "created_at": "2024-01-15T10:30:00",
    "updated_at": "2024-01-15T10:30:00"
  }
}
```

**Validações:**

- `model`: Mínimo 2 caracteres
- `color`: Mínimo 2 caracteres
- `plate`: 7-10 caracteres, único
- `factory_year`, `model_year`: 1900-2030
- `price`: Maior que zero
- `brand_id`: Deve existir

### Listar Carros

```http
GET /api/v1/cars/?offset=0&limit=10&search=corolla&brand_id=1&fuel_type=flex&transmission=automatic&is_available=true&min_price=100000&max_price=200000
Authorization: Bearer <token>
```

**Query Parameters:**

| Parâmetro | Tipo | Padrão | Descrição |
|-----------|------|--------|-----------|
| `offset` | int | 0 | Registros para pular |
| `limit` | int | 100 | Limite de registros (1-100) |
| `search` | string | null | Buscar por modelo ou placa |
| `brand_id` | int | null | Filtrar por marca |
| `fuel_type` | enum | null | Filtrar por combustível |
| `transmission` | enum | null | Filtrar por transmissão |
| `is_available` | bool | null | Filtrar por disponibilidade |
| `min_price` | float | null | Preço mínimo |
| `max_price` | float | null | Preço máximo |

**Response (200 OK):**

```json
{
  "cars": [
    {
      "id": 1,
      "model": "Corolla XEi",
      "factory_year": 2024,
      "model_year": 2024,
      "color": "Prata",
      "plate": "ABC1D23",
      "fuel_type": "flex",
      "transmission": "automatic",
      "price": 149900.00,
      "description": "Carro em perfeito estado",
      "is_available": true,
      "brand_id": 1,
      "owner_id": 1,
      "created_at": "2024-01-15T10:30:00",
      "updated_at": "2024-01-15T10:30:00",
      "brand": {...},
      "owner": {...}
    }
  ],
  "offset": 0,
  "limit": 10
}
```

### Buscar Carro por ID

```http
GET /api/v1/cars/{car_id}
Authorization: Bearer <token>
```

**Response (200 OK):**

Retorna o carro completo com marca e proprietário.

**Response (403 Forbidden):**

```json
{
  "detail": "Not enough permissions to access this car"
}
```

**Response (404 Not Found):**

```json
{
  "detail": "Carro não encontrado"
}
```

### Atualizar Carro

```http
PUT /api/v1/cars/{car_id}
Authorization: Bearer <token>
Content-Type: application/json
```

**Request Body (todos opcionais):**

```json
{
  "model": "Corolla XEi 2.0",
  "color": "Branco",
  "price": 154900.00,
  "is_available": false
}
```

**Response (200 OK):**

Retorna o carro atualizado com marca e proprietário.

### Deletar Carro

```http
DELETE /api/v1/cars/{car_id}
Authorization: Bearer <token>
```

**Response (204 No Content)**

## 📊 Códigos de Status HTTP

| Código | Descrição | Quando é Retornado |
|--------|-----------|-------------------|
| `200 OK` | Sucesso | Operações de leitura e atualização |
| `201 Created` | Criado | Criação de recursos |
| `204 No Content` | Sem conteúdo | Exclusão bem-sucedida |
| `400 Bad Request` | Requisição inválida | Validação falhou, recurso duplicado |
| `401 Unauthorized` | Não autorizado | Token ausente, inválido ou expirado |
| `403 Forbidden` | Proibido | Sem permissão para acessar recurso |
| `404 Not Found` | Não encontrado | Recurso não existe |
| `500 Internal Server Error` | Erro interno | Erro não esperado no servidor |

## 🔒 Autenticação

Para endpoints protegidos, inclua o header:

```
Authorization: Bearer <seu_token_jwt>
```

### Exemplo com cURL

```bash
# Criar carro
curl -X POST "http://localhost:8000/api/v1/cars/" \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..." \
  -H "Content-Type: application/json" \
  -d '{"model":"Corolla","factory_year":2024,"model_year":2024,"color":"Prata","plate":"ABC1D23","fuel_type":"flex","transmission":"automatic","price":149900,"brand_id":1}'
```

### Exemplo com Python (requests)

```python
import requests

token = "seu_token_jwt"
headers = {"Authorization": f"Bearer {token}"}

response = requests.get(
    "http://localhost:8000/api/v1/cars/",
    headers=headers
)

print(response.json())
```

## 📚 Documentação Interativa

Com o servidor rodando, acesse:

- **Swagger UI**: http://localhost:8000/docs
- **ReDoc**: http://localhost:8000/redoc

## 🚀 Próximo Passo

Com os endpoints conhecidos, prossiga para [Modelagem do Sistema](system-modeling.md).

---

**Dúvidas?** Consulte [Autenticação e Segurança](authentication.md) para detalhes sobre JWT.
