# Modelagem do Sistema

Este documento apresenta a modelagem completa do sistema, incluindo modelos de dados, arquitetura e fluxos principais.

## 📊 Modelos de Dados (ERD)

### Diagrama Entidade-Relacionamento

```mermaid
erDiagram
    USER ||--o{ CAR : owns
    BRAND ||--o{ CAR : "has many"
    
    USER {
        int id PK
        string username UK
        string password
        string email UK
        datetime created_at
        datetime updated_at
    }
    
    BRAND {
        int id PK
        string name UK
        string description
        bool is_active
        datetime created_at
        datetime updated_at
    }
    
    CAR {
        int id PK
        string model
        int factory_year
        int model_year
        string color
        string plate UK
        string fuel_type
        string transmission
        decimal price
        string description
        bool is_available
        int brand_id FK
        int owner_id FK
        datetime created_at
        datetime updated_at
    }
```

### Detalhes dos Modelos

#### User (Usuário)

| Campo | Tipo | Restrições | Descrição |
|-------|------|------------|-----------|
| `id` | int | PRIMARY KEY, AUTO INCREMENT | Identificador único |
| `username` | string(50) | UNIQUE, NOT NULL | Nome de usuário |
| `password` | string | NOT NULL | Senha com hash (Argon2) |
| `email` | string(100) | UNIQUE, NOT NULL | Email do usuário |
| `created_at` | datetime | DEFAULT NOW() | Data de criação |
| `updated_at` | datetime | DEFAULT NOW(), ON UPDATE | Data de atualização |

#### Brand (Marca)

| Campo | Tipo | Restrições | Descrição |
|-------|------|------------|-----------|
| `id` | int | PRIMARY KEY, AUTO INCREMENT | Identificador único |
| `name` | string(50) | UNIQUE, NOT NULL | Nome da marca |
| `description` | text | NULLABLE | Descrição da marca |
| `is_active` | bool | DEFAULT TRUE | Status da marca |
| `created_at` | datetime | DEFAULT NOW() | Data de criação |
| `updated_at` | datetime | DEFAULT NOW(), ON UPDATE | Data de atualização |

#### Car (Carro)

| Campo | Tipo | Restrições | Descrição |
|-------|------|------------|-----------|
| `id` | int | PRIMARY KEY, AUTO INCREMENT | Identificador único |
| `model` | string(100) | NOT NULL | Modelo do veículo |
| `factory_year` | int | NOT NULL | Ano de fabricação |
| `model_year` | int | NOT NULL | Ano modelo |
| `color` | string(30) | NOT NULL | Cor do veículo |
| `plate` | string(10) | UNIQUE, INDEX | Placa do veículo |
| `fuel_type` | enum | NOT NULL | Tipo de combustível |
| `transmission` | enum | NOT NULL | Tipo de transmissão |
| `price` | decimal(10,2) | NOT NULL | Preço do veículo |
| `description` | text | NULLABLE | Descrição detalhada |
| `is_available` | bool | DEFAULT TRUE | Disponibilidade |
| `brand_id` | int | FOREIGN KEY → Brand.id | Marca do veículo |
| `owner_id` | int | FOREIGN KEY → User.id | Proprietário |
| `created_at` | datetime | DEFAULT NOW() | Data de criação |
| `updated_at` | datetime | DEFAULT NOW(), ON UPDATE | Data de atualização |

### Enums

#### FuelType (Tipo de Combustível)

```mermaid
flowchart TD
    FuelType[FuelType] --> GASOLINE[gasoline]
    FuelType --> ETHANOL[ethanol]
    FuelType --> FLEX[flex]
    FuelType --> DIESEL[diesel]
    FuelType --> ELECTRIC[electric]
    FuelType --> HYBRID[hybrid]
```

#### TransmissionType (Tipo de Transmissão)

```mermaid
flowchart TD
    TransmissionType[TransmissionType] --> MANUAL[manual]
    TransmissionType --> AUTOMATIC[automatic]
    TransmissionType --> SEMI_AUTOMATIC[semi_automatic]
    TransmissionType --> CVT[cvt]
```

### Relacionamentos

```mermaid
flowchart LR
    subgraph "Relacionamento 1:N"
        USER[User] -->|owns| CARS[Cars]
        BRAND[Brand] -->|has| CARS
    end
    
    style USER fill:#e1f5ff
    style BRAND fill:#e1f5ff
    style CARS fill:#fff4e1
```

## 🏗️ Arquitetura do Sistema

### Visão Geral da Arquitetura

```mermaid
flowchart TB
    subgraph "Cliente"
        WEB[Web Browser]
        MOBILE[Mobile App]
        API_CLIENT[API Client]
    end
    
    subgraph "API Layer"
        FASTAPI[FastAPI Application]
        ROUTERS[Routers]
        MIDDLEWARE[Middleware]
    end
    
    subgraph "Business Layer"
        SECURITY[Security Module]
        SCHEMAS[Pydantic Schemas]
    end
    
    subgraph "Data Layer"
        MODELS[SQLAlchemy Models]
        DATABASE[(SQLite Database)]
    end
    
    WEB & MOBILE & API_CLIENT -->|HTTP/HTTPS| FASTAPI
    FASTAPI --> ROUTERS
    ROUTERS --> MIDDLEWARE
    MIDDLEWARE --> SECURITY
    SECURITY --> SCHEMAS
    SCHEMAS --> MODELS
    MODELS --> DATABASE
```

### Arquitetura em Camadas

```mermaid
flowchart TB
    subgraph "Camada de Apresentação"
        A1[Routers - Auth]
        A2[Routers - Users]
        A3[Routers - Cars]
        A4[Routers - Brands]
    end
    
    subgraph "Camada de Segurança"
        B1[JWT Authentication]
        B2[Password Hashing]
        B3[Ownership Validation]
    end
    
    subgraph "Camada de Validação"
        C1[Request Schemas]
        C2[Response Schemas]
        C3[Field Validators]
    end
    
    subgraph "Camada de Dados"
        D1[User Model]
        D2[Car Model]
        D3[Brand Model]
        D4[Database Session]
    end
    
    A1 & A2 & A3 & A4 --> B1 & B2 & B3
    B1 & B2 & B3 --> C1 & C2 & C3
    C1 & C2 & C3 --> D1 & D2 & D3 & D4
```

### Fluxo de Requisição

```mermaid
sequenceDiagram
    participant C as Cliente
    participant R as Router
    participant S as Security
    participant SC as Schema
    participant M as Model
    participant DB as Database
    
    C->>R: HTTP Request
    R->>S: Validate Token
    S-->>R: User Context
    R->>SC: Validate Input
    SC-->>R: Validated Data
    R->>M: Map to Model
    M->>DB: Execute Query
    DB-->>M: Result
    M-->>R: ORM Objects
    R->>SC: Serialize Response
    SC-->>R: JSON Response
    R-->>C: HTTP Response
```

## 🔐 Fluxo de Autenticação

### Login e Geração de Token

```mermaid
sequenceDiagram
    participant U as Usuário
    participant API as API
    participant DB as Database
    participant JWT as JWT Module
    
    U->>API: POST /auth/token (email, password)
    API->>DB: SELECT user WHERE email = ?
    DB-->>API: User data (with hashed password)
    API->>API: Verify password (Argon2)
    
    alt Senha válida
        API->>JWT: Create access token (user_id, exp)
        JWT-->>API: Encoded JWT token
        API-->>U: 200 OK {access_token, token_type}
    else Senha inválida
        API-->>U: 401 Unauthorized
    end
```

### Acesso a Endpoint Protegido

```mermaid
sequenceDiagram
    participant U as Usuário
    participant API as API
    participant JWT as JWT Module
    participant DB as Database
    
    U->>API: GET /cars/ (Authorization: Bearer token)
    API->>JWT: Decode and verify token
    JWT-->>API: Payload (user_id, exp)
    
    alt Token válido e não expirado
        API->>DB: SELECT user WHERE id = user_id
        DB-->>API: User data
        API-->>U: 200 OK (cars list)
    else Token inválido/expirado
        API-->>U: 401 Unauthorized
    end
```

### Refresh Token

```mermaid
flowchart TD
    A[Usuário com token expirado] --> B[POST /auth/refresh_token]
    B --> C{Token atual válido?}
    C -->|Sim| D[Gerar novo access token]
    C -->|Não| E[401 Unauthorized]
    D --> F[Retornar novo token]
    F --> G[Usuário usa novo token]
    E --> H[Usuário faz login novamente]
```

### Diagrama Completo de Autenticação

```mermaid
flowchart TB
    subgraph "Registro"
        A1[Usuário envia dados] --> A2[Validar dados]
        A2 --> A3[Hash da senha]
        A3 --> A4[Salvar no banco]
        A4 --> A5[Retornar usuário criado]
    end
    
    subgraph "Login"
        B1[Usuário envia credenciais] --> B2[Buscar usuário]
        B2 --> B3[Verificar senha]
        B3 --> B4[Gerar JWT token]
        B4 --> B5[Retornar token]
    end
    
    subgraph "Acesso Protegido"
        C1[Requisição com token] --> C2[Validar token]
        C2 --> C3[Buscar usuário]
        C3 --> C4[Executar ação]
        C4 --> C5[Retornar resposta]
    end
    
    A5 -.-> B1
    B5 -.-> C1
```

## 🚗 Fluxo CRUD de Carros

### Criar Carro

```mermaid
sequenceDiagram
    participant U as Usuário Autenticado
    participant API as API
    participant DB as Database
    
    U->>API: POST /cars/ (car data + token)
    API->>API: Validar token JWT
    API->>DB: Verificar placa duplicada
    DB-->>API: Resultado
    
    alt Placa já existe
        API-->>U: 400 Bad Request
    else Placa disponível
        API->>DB: Verificar marca existe
        DB-->>API: Resultado
        
        alt Marca não existe
            API-->>U: 400 Bad Request
        else Marca existe
            API->>DB: INSERT car (owner_id = user.id)
            DB-->>API: Carro criado
            API-->>U: 201 Created (carro completo)
        end
    end
```

### Listar Carros

```mermaid
flowchart TD
    A[GET /cars/ com filtros] --> B[Validar token JWT]
    B --> C[Construir query base]
    C --> D{Filtro search?}
    D -->|Sim| E[WHERE model/plate LIKE]
    D -->|Não| F[Próximo filtro]
    E --> F
    F --> G{Filtro brand_id?}
    G -->|Sim| H[WHERE brand_id = ?]
    G -->|Não| I{Filtro fuel_type?}
    H --> I
    I --> J{Filtro transmission?}
    J -->|Sim| K[WHERE transmission = ?]
    J -->|Não| L{Filtro is_available?}
    K --> L
    L --> M{Filtro price range?}
    M -->|Sim| N[WHERE price BETWEEN]
    M -->|Não| O[Aplicar OFFSET/LIMIT]
    N --> O
    O --> P[Executar query]
    P --> Q[Retornar lista com brand e owner]
```

### Atualizar Carro

```mermaid
sequenceDiagram
    participant U as Usuário
    participant API as API
    participant DB as Database
    
    U->>API: PUT /cars/{id} (data + token)
    API->>API: Validar token
    API->>DB: Buscar carro por ID
    DB-->>API: Carro (com owner_id)
    
    alt Carro não existe
        API-->>U: 404 Not Found
    else Carro existe
        API->>API: Verificar ownership
        alt Usuário != owner
            API-->>U: 403 Forbidden
        else Usuário == owner
            API->>DB: Verificar placa (se alterada)
            DB-->>API: Resultado
            
            alt Placa duplicada
                API-->>U: 400 Bad Request
            else Placa OK
                API->>DB: UPDATE car
                DB-->>API: Carro atualizado
                API-->>U: 200 OK (carro atualizado)
            end
        end
    end
```

### Deletar Carro

```mermaid
flowchart TD
    A[DELETE /cars/{id}] --> B[Validar token JWT]
    B --> C[Bucar carro no banco]
    C --> D{Carro existe?}
    D -->|Não| E[404 Not Found]
    D -->|Sim| F{Usuário é owner?}
    F -->|Não| G[403 Forbidden]
    F -->|Sim| H[DELETE do banco]
    H --> I[Commit transação]
    I --> J[204 No Content]
```

### Fluxo Completo CRUD

```mermaid
flowchart LR
    subgraph "CRUD Carros"
        C[Create] -->|POST /cars/| C1[Validar dados<br/>Verificar placa<br/>Salvar]
        R[Read] -->|GET /cars/| R1[Listar com filtros<br/>GET /cars/{id}<br/>Buscar por ID]
        U[Update] -->|PUT /cars/{id}| U1[Verificar ownership<br/>Validar dados<br/>Atualizar]
        D[Delete] -->|DELETE /cars/{id}| D1[Verificar ownership<br/>Remover]
    end
    
    style C fill:#e8f5e9
    style R fill:#e3f2fd
    style U fill:#fff3e0
    style D fill:#ffebee
```

## 🔒 Fluxo de Segurança

### Hash de Senha

```mermaid
flowchart TD
    A[Senha em texto plano] --> B[get_password_hash]
    B --> C[Gerar salt aleatório]
    C --> D[Aplicar Argon2]
    D --> E[Armazenar hash]
    
    style A fill:#ffebee
    style E fill:#e8f5e9
```

### Verificação de Senha

```mermaid
flowchart TD
    A[Login attempt] --> B[Buscar usuário no DB]
    B --> C{Usuário existe?}
    C -->|Não| D[Retornar erro genérico]
    C -->|Sim| E[verify_password]
    E --> F[Comparar hash com Argon2]
    F --> G{Senha válida?}
    G -->|Sim| H[Prosseguir autenticação]
    G -->|Não| I[Retornar erro genérico]
    
    style D fill:#fff3e0
    style I fill:#fff3e0
```

### Validação de Ownership

```mermaid
sequenceDiagram
    participant U as Usuário
    participant API as API
    participant R as Resource
    
    U->>API: Requisição para recurso protegido
    API->>API: Extrair user_id do token
    API->>API: Buscar owner_id do recurso
    API->>API: Comparar user_id == owner_id
    
    alt user_id == owner_id
        API->>R: Permitir operação
    else user_id != owner_id
        API->>U: 403 Forbidden
    end
```

### Ciclo de Vida do Token JWT

```mermaid
flowchart TD
    A[Login bem-sucedido] --> B[Gerar JWT token]
    B --> C[Token válido por 30 min]
    C --> D{Token expirou?}
    D -->|Não| E[Usar normalmente]
    D -->|Sim| F{Fazer refresh?}
    F -->|Sim| G[POST /auth/refresh_token]
    G --> H[Novo token gerado]
    H --> C
    F -->|Não| I[Fazer login novamente]
    I --> B
    
    style A fill:#e8f5e9
    style G fill:#e3f2fd
    style I fill:#fff3e0
```

### Matriz de Segurança

```mermaid
flowchart TB
    subgraph "Camadas de Segurança"
        A1[HTTPS/TLS] --> A2[JWT Authentication]
        A2 --> A3[Password Hashing Argon2]
        A3 --> A4[Ownership Validation]
        A4 --> A5[Input Validation]
        A5 --> A6[SQL Injection Prevention]
    end
    
    subgraph "Proteções"
        B1[Rate Limiting]
        B2[CORS Policy]
        B3[Error Handling]
        B4[Logging Seguro]
    end
    
    A6 --> B1 & B2 & B3 & B4
```

## 📈 Escalabilidade

### Arquitetura para Escala

```mermaid
flowchart TB
    subgraph "Load Balancer"
        LB[NGINX / HAProxy]
    end
    
    subgraph "API Instances"
        API1[FastAPI Instance 1]
        API2[FastAPI Instance 2]
        API3[FastAPI Instance 3]
    end
    
    subgraph "Database"
        DB[(PostgreSQL Cluster)]
    end
    
    subgraph "Cache"
        CACHE[(Redis)]
    end
    
    LB --> API1 & API2 & API3
    API1 & API2 & API3 --> DB
    API1 & API2 & API3 --> CACHE
```

## 🚀 Próximo Passo

Com a modelagem compreendida, prossiga para [Autenticação e Segurança](authentication.md).

---

**Dúvidas?** Consulte [API Endpoints](api-endpoints.md) para ver como os endpoints se relacionam com os modelos.
