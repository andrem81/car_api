# Release Notes

Histórico de versões e mudanças da Fastcar API.

## 📋 Versões

### [0.1.0] - 2024-01-15

#### ✨ Novas Funcionalidades

**Autenticação**
- Implementado sistema de autenticação JWT
- Endpoint de login (`POST /api/v1/auth/token`)
- Endpoint de refresh token (`POST /api/v1/auth/refresh_token`)
- Hash de senhas com Argon2

**Usuários**
- CRUD completo de usuários
- Validação de username e email únicos
- Validação de força de senha (mínimo 6 caracteres)
- Listagem com paginação e busca

**Carros**
- CRUD completo de carros
- Validação de propriedade (ownership)
- Filtros avançados (marca, combustível, transmissão, preço)
- Validação de placa única
- Enums para tipo de combustível e transmissão

**Marcas**
- CRUD completo de marcas
- Validação de nome único
- Status ativo/inativo
- Prevenção de exclusão de marcas com carros vinculados

**Infraestrutura**
- Configuração com SQLAlchemy Async
- Migrações com Alembic
- Validação de dados com Pydantic v2
- Documentação automática com Swagger UI e ReDoc

#### 🔒 Segurança

- Autenticação JWT com expiração configurável
- Hash de senhas com Argon2
- Validação de ownership em operações sensíveis
- Headers de segurança

#### 📊 Modelos de Dados

**User**
- id, username, email, password
- created_at, updated_at
- Relacionamento com Carros

**Brand**
- id, name, description, is_active
- created_at, updated_at
- Relacionamento com Carros

**Car**
- id, model, factory_year, model_year, color, plate
- fuel_type, transmission, price, description, is_available
- brand_id (FK), owner_id (FK)
- created_at, updated_at
- Relacionamento com Brand e User

#### 🧪 Testes

- Estrutura de testes com pytest
- Fixtures para banco de dados e autenticação
- Testes unitários e de integração

#### 📝 Documentação

- Documentação completa com MkDocs
- Guia de instalação e configuração
- Documentação de API endpoints
- Diagramas de arquitetura e fluxo

#### 🛠️ Ferramentas

- Ruff para linting e formatação
- Poetry para gerenciamento de dependências
- Taskipy para tasks de desenvolvimento
- MkDocs Material para documentação

---

## 🔮 Roadmap

### Versão 0.2.0 (Planejado)

**Funcionalidades**
- [ ] Upload de imagens para carros
- [ ] Sistema de favoritos
- [ ] Histórico de visualizações
- [ ] Exportação de dados (CSV, JSON)

**Melhorias**
- [ ] Cache com Redis
- [ ] Rate limiting avançado
- [ ] Webhooks para eventos
- [ ] GraphQL API

**Infraestrutura**
- [ ] Suporte a PostgreSQL
- [ ] Docker Compose para produção
- [ ] CI/CD pipeline
- [ ] Monitoramento com Prometheus

### Versão 0.3.0 (Planejado)

**Funcionalidades**
- [ ] Sistema de avaliações/reviews
- [ ] Chat entre usuários
- [ ] Notificações por email
- [ ] Dashboard de estatísticas

**Segurança**
- [ ] 2FA (Two-Factor Authentication)
- [ ] OAuth2 (Google, GitHub login)
- [ ] Audit logs
- [ ] GDPR compliance

---

## 📝 Changelog Completo

### [0.1.0] - 2024-01-15

#### Added

- **Auth**
  - `POST /api/v1/auth/token` - Gerar token de acesso
  - `POST /api/v1/auth/refresh_token` - Atualizar token

- **Users**
  - `POST /api/v1/users/` - Criar usuário
  - `GET /api/v1/users/` - Listar usuários
  - `GET /api/v1/users/{user_id}` - Buscar usuário
  - `PUT /api/v1/users/{user_id}` - Atualizar usuário
  - `DELETE /api/v1/users/{user_id}` - Deletar usuário

- **Brands**
  - `POST /api/v1/brands/` - Criar marca
  - `GET /api/v1/brands/` - Listar marcas
  - `GET /api/v1/brands/{brand_id}` - Buscar marca
  - `PUT /api/v1/brands/{brand_id}` - Atualizar marca
  - `DELETE /api/v1/brands/{brand_id}` - Deletar marca

- **Cars**
  - `POST /api/v1/cars/` - Criar carro
  - `GET /api/v1/cars/` - Listar carros
  - `GET /api/v1/cars/{car_id}` - Buscar carro
  - `PUT /api/v1/cars/{car_id}` - Atualizar carro
  - `DELETE /api/v1/cars/{car_id}` - Deletar carro

- **Health Check**
  - `GET /health_check` - Verificação de saúde

- **Models**
  - User, Brand, Car models com SQLAlchemy
  - Enums FuelType e TransmissionType

- **Schemas**
  - Request/Response schemas com Pydantic
  - Validações de campo personalizadas

- **Core**
  - Database configuration (AsyncSession)
  - Security module (JWT, password hashing)
  - Settings management

- **Migrations**
  - Alembic configuration
  - Initial migration para tabelas

- **Documentation**
  - MkDocs com Material theme
  - Swagger UI e ReDoc automáticos

#### Changed

- Nada (versão inicial)

#### Deprecated

- Nada

#### Removed

- Nada

#### Fixed

- Nada (versão inicial)

#### Security

- JWT authentication
- Argon2 password hashing
- Ownership validation
- Input validation com Pydantic

---

## 📊 Estatísticas do Projeto

| Métrica | Valor |
|---------|-------|
| Versão Atual | 0.1.0 |
| Data de Lançamento | 2024-01-15 |
| Endpoints | 17 |
| Modelos | 3 |
| Schemas | 12 |
| Python | >=3.13 |
| Dependências | 11 |

---

## 🔗 Links

- [Repositório](https://github.com/usuario/car_api)
- [Issues](https://github.com/usuario/car_api/issues)
- [Discussions](https://github.com/usuario/car_api/discussions)
- [Documentação](https://usuario.github.io/car_api/)

---

## 📞 Contato

Para dúvidas sobre releases, abra uma issue ou discussion no GitHub.
