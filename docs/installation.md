# Instalação

Este guia fornece instruções passo a passo para instalar e configurar a Fastcar API em seu ambiente de desenvolvimento.

## 📥 Clone do Repositório

Primeiro, clone o repositório do projeto:

```bash
git clone <url-do-repositorio>
cd car_api
```

## 📦 Instalação das Dependências

### 1. Instalar Dependências com Poetry

O Poetry gerencia todas as dependências do projeto:

```bash
# Instalar dependências principais
poetry install

# Instalar dependências principais + dependências de desenvolvimento
poetry install --with dev
```

### 2. Verificar Instalação

Após a instalação, verifique se tudo está correto:

```bash
# Listar pacotes instalados
poetry show

# Verificar ambiente
poetry env info
```

## 🔧 Configuração Inicial

### 1. Criar Arquivo de Variáveis de Ambiente

Copie o arquivo de exemplo (se existir) ou crie um novo:

```bash
# O arquivo .env já deve existir no projeto
# Caso contrário, crie baseado no modelo abaixo
```

### 2. Configurar Variáveis de Ambiente

Edite o arquivo `.env` na raiz do projeto:

```bash
# .env
DATABASE_URL='sqlite+aiosqlite:///car.db'
JWT_SECRET_KEY='sua-chave-secreta-aqui'
JWT_ALGORITHM='HS256'
JWT_EXPIRATION_MINUTES=30
```

> **⚠️ Importante**: Em produção, gere uma `JWT_SECRET_KEY` segura:

```bash
# Gerar chave secreta segura
python -c "import secrets; print(secrets.token_urlsafe(64))"
```

### 3. Executar Migrações do Banco de Dados

Execute as migrações para criar as tabelas:

```bash
# Ativar ambiente virtual do Poetry
poetry shell

# Executar migrações
alembic upgrade head
```

## 🏃 Executando a Aplicação

### Modo Desenvolvimento

Utilize o taskipy para rodar o servidor em modo desenvolvimento:

```bash
# Via Poetry task
poetry run task run

# Ou diretamente com FastAPI
poetry run fastapi dev car_api/app.py
```

O servidor será iniciado em: `http://127.0.0.1:8000`

### Acessando a Documentação

Com o servidor rodando, acesse:

- **Swagger UI**: http://127.0.0.1:8000/docs
- **ReDoc**: http://127.0.0.1:8000/redoc
- **Health Check**: http://127.0.0.1:8000/health_check

## 🧪 Verificando a Instalação

### 1. Testar Health Check

```bash
curl http://127.0.0.1:8000/health_check
```

Resposta esperada:

```json
{
  "status": "ok"
}
```

### 2. Executar Testes (Opcional)

```bash
# Executar testes com pytest
poetry run pytest tests/
```

## 🐳 Instalação com Docker (Opcional)

Se preferir usar Docker:

### 1. Criar Dockerfile

```dockerfile
FROM python:3.13-slim

WORKDIR /app

# Instalar Poetry
RUN pip install poetry

# Copiar arquivos de dependência
COPY pyproject.toml poetry.lock ./

# Instalar dependências
RUN poetry config virtualenvs.create false && poetry install --no-interaction --no-ansi

# Copiar código fonte
COPY . .

# Expor porta
EXPOSE 8000

# Comando para rodar
CMD ["fastapi", "run", "car_api/app.py", "--port", "8000"]
```

### 2. Construir e Rodar

```bash
# Construir imagem
docker build -t fastcar-api .

# Rodar container
docker run -p 8000:8000 -v $(pwd)/car.db:/app/car.db fastcar-api
```

## 🔍 Solução de Problemas

### Erro: "No module named 'aiosqlite'"

```bash
# Reinstalar dependências
poetry install --no-cache
```

### Erro: "DATABASE_URL not configured"

Verifique se o arquivo `.env` existe e está configurado corretamente:

```bash
# Verificar arquivo
cat .env
```

### Erro: "Table already exists"

Resetar o banco de dados:

```bash
# Remover banco existente
rm car.db

# Executar migrações novamente
alembic upgrade head
```

### Erro: "Poetry command not found"

Verifique a instalação do Poetry:

```bash
# Verificar se Poetry está no PATH
which poetry

# Se não estiver, adicione ao PATH
export PATH="$HOME/.local/bin:$PATH"
```

## 📋 Resumo dos Comandos

```bash
# 1. Clone o repositório
git clone <url> && cd car_api

# 2. Instale dependências
poetry install --with dev

# 3. Configure .env (já existe no projeto)

# 4. Execute migrações
poetry shell
alembic upgrade head

# 5. Rode a aplicação
poetry run task run
```

## 🚀 Próximo Passo

Com a instalação concluída, prossiga para a [Configuração do Projeto](configuration.md).

---

**Precisa de ajuda?** Consulte [Desenvolvimento](development.md) para mais detalhes sobre o ambiente de desenvolvimento.
