# Fastcar API - Documentação

Bem-vindo à documentação oficial da **Fastcar API**, uma API REST completa para gerenciamento de carros e usuários.

## 🚀 Sobre o Projeto

A Fastcar API é uma aplicação backend moderna construída com **FastAPI** e **SQLAlchemy**, projetada para fornecer uma solução robusta e escalável para gerenciamento de veículos. A API permite o cadastro, listagem, atualização e exclusão de carros, marcas e usuários, com autenticação segura baseada em JWT.

## 📋 Recursos Principais

- **Autenticação JWT**: Sistema seguro de autenticação com tokens de acesso
- **CRUD Completo**: Operações de Create, Read, Update e Delete para carros, marcas e usuários
- **Filtros Avançados**: Busca e filtragem de carros por múltiplos critérios
- **Validação de Dados**: Validações robustas utilizando Pydantic
- **Banco de Dados Assíncrono**: Suporte a operações assíncronas com SQLAlchemy e SQLite
- **Documentação Automática**: Swagger UI e ReDoc integrados
- **Migrações**: Controle de versão de banco de dados com Alembic

## 🔗 Links Rápidos

| Seção | Descrição |
|-------|-----------|
| [Visão Geral](overview.md) | Entenda o propósito e arquitetura do projeto |
| [Instalação](installation.md) | Guia passo a passo para instalar o projeto |
| [Configuração](configuration.md) | Como configurar variáveis de ambiente e parâmetros |
| [API Endpoints](api-endpoints.md) | Documentação completa dos endpoints |
| [Modelagem](system-modeling.md) | Diagramas e modelagem do sistema |

## 🛠️ Tecnologias Utilizadas

- **Framework**: FastAPI
- **Banco de Dados**: SQLite (com SQLAlchemy Async)
- **ORM**: SQLAlchemy 2.0+
- **Validação**: Pydantic v2
- **Autenticação**: JWT (PyJWT)
- **Hash de Senha**: pwdlib (Argon2)
- **Migrações**: Alembic
- **Gerenciador de Pacotes**: Poetry
- **Linting/Formatting**: Ruff
- **Documentação**: MkDocs com Material Theme

## 📦 Status do Projeto

| Versão | Status |
|--------|--------|
| 0.1.0 | ✅ Estável |

## 🤝 Contribuição

Contribuições são bem-vindas! Consulte nosso guia de [contribuição](contributing.md) para mais informações.

## 📄 Licença

Este projeto está sob licença MIT.

---

**Precisa de ajuda?** Consulte as seções de [desenvolvimento](development.md) ou [testes](testing.md) para mais detalhes.
