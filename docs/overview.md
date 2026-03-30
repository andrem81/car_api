# Visão Geral do Projeto

## 📌 Propósito

A **Fastcar API** é uma API RESTful desenvolvida para fornecer uma solução completa de gerenciamento de veículos. O sistema permite que usuários se cadastrem, autentiquem e gerenciem seus carros, incluindo informações detalhadas sobre marca, modelo, ano, preço e características técnicas.

## 🎯 Objetivos

- Fornecer uma API robusta e escalável para gerenciamento de carros
- Implementar autenticação segura com JWT
- Oferecer endpoints RESTful seguindo boas práticas
- Garantir validação de dados consistente
- Proporcionar uma experiência de desenvolvimento agradável com documentação clara

## 🏗️ Arquitetura

A API segue a arquitetura **REST** com os seguintes princípios:

### Camadas da Aplicação

```
┌─────────────────────────────────────────┐
│           Cliente (Frontend)            │
└─────────────────┬───────────────────────┘
                  │ HTTP/HTTPS
                  ▼
┌─────────────────────────────────────────┐
│            API Gateway / Load           │
│            Balancer (Opcional)          │
└─────────────────┬───────────────────────┘
                  │
                  ▼
┌─────────────────────────────────────────┐
│          Camada de Routers              │
│      (Endpoints da API - FastAPI)       │
└─────────────────┬───────────────────────┘
                  │
                  ▼
┌─────────────────────────────────────────┐
│         Camada de Segurança             │
│    (Autenticação JWT, Validações)       │
└─────────────────┬───────────────────────┘
                  │
                  ▼
┌─────────────────────────────────────────┐
│         Camada de Schemas               │
│      (Pydantic - Validação/Serial.)     │
└─────────────────┬───────────────────────┘
                  │
                  ▼
┌─────────────────────────────────────────┐
│         Camada de Modelos               │
│      (SQLAlchemy - ORM Models)          │
└─────────────────┬───────────────────────┘
                  │
                  ▼
┌─────────────────────────────────────────┐
│         Camada de Database              │
│      (SQLite com AsyncSession)          │
└─────────────────────────────────────────┘
```

## 📊 Funcionalidades Principais

### Autenticação e Usuários

- **Registro de Usuário**: Criação de contas com validação de email e username
- **Login**: Autenticação via email/senha com retorno de token JWT
- **Refresh Token**: Renovação de tokens de acesso
- **CRUD de Usuários**: Gerenciamento completo de contas

### Gestão de Carros

- **Cadastro de Carros**: Registro com informações detalhadas
- **Listagem com Filtros**: Busca por modelo, placa, marca, combustível, transmissão, preço
- **Atualização**: Edição de informações dos veículos
- **Exclusão**: Remoção de carros do sistema
- **Validação de Propriedade**: Apenas o dono pode editar/excluir seu carro

### Gestão de Marcas

- **Cadastro de Marcas**: Registro de fabricantes de veículos
- **Listagem**: Visualização de todas as marcas
- **Filtros**: Busca por nome e status (ativo/inativo)
- **Validação**: Impede exclusão de marcas com carros vinculados

## 🔐 Segurança

- **Hash de Senhas**: Utilização de Argon2 para hashing seguro
- **JWT Tokens**: Autenticação stateless com expiração configurável
- **Validação de Propriedade**: Verificação de ownership em operações sensíveis
- **HTTPS Ready**: Preparado para deploy com SSL/TLS

## 📈 Escalabilidade

A arquitetura foi projetada para escalar horizontalmente:

- **Stateless**: Tokens JWT permitem escalabilidade sem sessões
- **Async/Await**: Operações assíncronas para melhor concorrência
- **Connection Pool**: Gerenciamento eficiente de conexões com o banco

## 🔧 Extensibilidade

O código segue princípios SOLID e padrões que facilitam:

- Adição de novos endpoints
- Implementação de novos modelos
- Integração com outros serviços
- Substituição do banco de dados (PostgreSQL, MySQL, etc.)

## 📝 Padrões de Código

- **Type Hints**: Tipagem estática em todo o código
- **Pydantic v2**: Validação de dados moderna e performática
- **SQLAlchemy 2.0**: Sintaxe moderna do ORM
- **Ruff**: Linting e formatação consistentes

---

Próxima seção: [Pré-requisitos](prerequisites.md)
