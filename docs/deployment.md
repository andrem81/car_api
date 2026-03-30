# Deploy

Este guia descreve os procedimentos e melhores práticas para deploy da Fastcar API em produção.

## 📋 Visão Geral

O deploy da Fastcar API pode ser realizado em diferentes ambientes e plataformas. Este documento cobre:

- Preparação para produção
- Deploy com Docker
- Deploy em plataformas cloud
- Configuração de produção
- Monitoramento e logs

## 🔧 Preparação para Produção

### Checklist Pré-Deploy

- [ ] Variáveis de ambiente configuradas
- [ ] JWT_SECRET_KEY segura e única
- [ ] Banco de dados de produção configurado
- [ ] HTTPS/TLS habilitado
- [ ] Logs configurados
- [ ] Backup de banco de dados automatizado
- [ ] Monitoramento configurado
- [ ] Rate limiting habilitado
- [ ] CORS configurado corretamente

### Variáveis de Ambiente de Produção

```bash
# .env.production

# Banco de dados (PostgreSQL recomendado)
DATABASE_URL='postgresql+asyncpg://user:password@db-host:5432/car_api_prod'

# JWT
JWT_SECRET_KEY='<gerar-chave-segura-de-64-caracteres>'
JWT_ALGORITHM='HS256'
JWT_EXPIRATION_MINUTES=30

# Produção
ENVIRONMENT='production'
DEBUG='false'
```

### Gerar Chave Secreta

```bash
# Python
python -c "import secrets; print(secrets.token_urlsafe(64))"

# OpenSSL
openssl rand -hex 64
```

## 🐳 Deploy com Docker

### Dockerfile de Produção

```dockerfile
# Dockerfile
FROM python:3.13-slim

# Definir variáveis de ambiente
ENV PYTHONDONTWRITEBYTECODE=1 \
    PYTHONUNBUFFERED=1 \
    PIP_NO_CACHE_DIR=1 \
    PIP_DISABLE_PIP_VERSION_CHECK=1

# Instalar dependências do sistema
RUN apt-get update && apt-get install -y --no-install-recommends \
    gcc \
    && rm -rf /var/lib/apt/lists/*

# Instalar Poetry
RUN pip install poetry==2.0.0

# Definir diretório de trabalho
WORKDIR /app

# Copiar arquivos de dependência
COPY pyproject.toml poetry.lock ./

# Instalar dependências
RUN poetry config virtualenvs.create false \
    && poetry install --only main --no-interaction --no-ansi

# Copiar código fonte
COPY . .

# Criar usuário não-root para segurança
RUN useradd --create-home --shell /bin/bash appuser \
    && chown -R appuser:appuser /app
USER appuser

# Expor porta
EXPOSE 8000

# Health check
HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 \
    CMD python -c "import urllib.request; urllib.request.urlopen('http://localhost:8000/health_check')" || exit 1

# Comando para rodar
CMD ["fastapi", "run", "car_api/app.py", "--port", "8000", "--host", "0.0.0.0"]
```

### Docker Compose

```yaml
# docker-compose.yml
version: '3.8'

services:
  api:
    build: .
    ports:
      - "8000:8000"
    environment:
      - DATABASE_URL=postgresql+asyncpg://user:password@db:5432/car_api
      - JWT_SECRET_KEY=${JWT_SECRET_KEY}
      - JWT_ALGORITHM=HS256
      - JWT_EXPIRATION_MINUTES=30
    depends_on:
      db:
        condition: service_healthy
    restart: unless-stopped
    networks:
      - car_api_network

  db:
    image: postgres:15-alpine
    environment:
      - POSTGRES_USER=user
      - POSTGRES_PASSWORD=password
      - POSTGRES_DB=car_api
    volumes:
      - postgres_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U user -d car_api"]
      interval: 10s
      timeout: 5s
      retries: 5
    restart: unless-stopped
    networks:
      - car_api_network

  migrations:
    build: .
    command: alembic upgrade head
    environment:
      - DATABASE_URL=postgresql+asyncpg://user:password@db:5432/car_api
    depends_on:
      db:
        condition: service_healthy
    networks:
      - car_api_network

volumes:
  postgres_data:

networks:
  car_api_network:
    driver: bridge
```

### Build e Run

```bash
# Build das imagens
docker-compose build

# Rodar serviços
docker-compose up -d

# Ver logs
docker-compose logs -f api

# Parar serviços
docker-compose down
```

## ☁️ Deploy em Plataformas Cloud

### AWS (Elastic Beanstalk)

```bash
# Instalar EB CLI
pip install awsebcli

# Inicializar aplicação
eb init

# Criar ambiente
eb create production

# Deploy
eb deploy

# Abrir aplicação
eb open
```

### Google Cloud Run

```bash
# Autenticar
gcloud auth login

# Configurar projeto
gcloud config set project PROJECT_ID

# Build e push da imagem
gcloud builds submit --tag gcr.io/PROJECT_ID/fastcar-api

# Deploy no Cloud Run
gcloud run deploy fastcar-api \
  --image gcr.io/PROJECT_ID/fastcar-api \
  --platform managed \
  --region us-central1 \
  --allow-unauthenticated
```

### Heroku

```bash
# Instalar Heroku CLI
# https://devcenter.heroku.com/articles/heroku-cli

# Login
heroku login

# Criar app
heroku create fastcar-api

# Adicionar buildpack Python
heroku buildpacks:set heroku/python

# Configurar variáveis de ambiente
heroku config:set JWT_SECRET_KEY=your-secret-key
heroku config:set DATABASE_URL=postgresql://...

# Deploy
git push heroku main

# Abrir app
heroku open
```

### Railway

```bash
# Instalar Railway CLI
npm install -g @railway/cli

# Login
railway login

# Inicializar projeto
railway init

# Deploy
railway up
```

## 🗄️ Banco de Dados em Produção

### PostgreSQL (Recomendado)

```bash
# Instalar driver
poetry add asyncpg

# Configurar DATABASE_URL
DATABASE_URL='postgresql+asyncpg://user:password@host:5432/database'
```

### MySQL

```bash
# Instalar driver
poetry add aiomysql

# Configurar DATABASE_URL
DATABASE_URL='mysql+aiomysql://user:password@host:3306/database'
```

### Migrações em Produção

```bash
# Rodar migrações
alembic upgrade head

# Verificar status
alembic current

# Rollback se necessário
alembic downgrade -1
```

## 🔒 Segurança em Produção

### HTTPS com Nginx

```nginx
# nginx.conf
server {
    listen 80;
    server_name api.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name api.example.com;

    ssl_certificate /etc/ssl/certs/cert.pem;
    ssl_certificate_key /etc/ssl/private/key.pem;

    location / {
        proxy_pass http://localhost:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

### Headers de Segurança

```python
# car_api/app.py
from fastapi import FastAPI, Request
from fastapi.responses import Response

app = FastAPI()

@app.middleware("http")
async def add_security_headers(request: Request, call_next):
    response = await call_next(request)
    response.headers["X-Content-Type-Options"] = "nosniff"
    response.headers["X-Frame-Options"] = "DENY"
    response.headers["X-XSS-Protection"] = "1; mode=block"
    response.headers["Strict-Transport-Security"] = "max-age=31536000; includeSubDomains"
    return response
```

### Rate Limiting

```bash
# Instalar slowapi
poetry add slowapi
```

```python
# car_api/app.py
from slowapi import SlowAPI, _rate_limit_exceeded_handler
from slowapi.util import get_remote_address
from slowapi.errors import RateLimitExceeded

app = FastAPI()
limiter = SlowAPI(key_func=get_remote_address)
app.state.limiter = limiter

app.add_exception_handler(RateLimitExceeded, _rate_limit_exceeded_handler)

@app.post("/api/v1/auth/token")
@limiter.limit("5/minute")
async def token(request: Request, ...):
    pass
```

## 📊 Monitoramento e Logs

### Logs Estruturados

```python
# car_api/core/logging_config.py
import logging
import sys

def setup_logging():
    logging.basicConfig(
        level=logging.INFO,
        format='%(asctime)s - %(name)s - %(levelname)s - %(message)s',
        handlers=[
            logging.StreamHandler(sys.stdout)
        ]
    )
```

### Health Check

```python
# car_api/app.py
@app.get('/health_check', status_code=200)
async def health_check():
    return {
        'status': 'ok',
        'version': '0.1.0'
    }

@app.get('/health_check/db')
async def health_check_db(db: AsyncSession = Depends(get_session)):
    try:
        await db.execute(select(1))
        return {'status': 'ok', 'database': 'connected'}
    except Exception as e:
        return {'status': 'error', 'database': str(e)}
```

### Métricas

```python
# Instalar prometheus-client
poetry add prometheus-client

# car_api/app.py
from prometheus_fastapi_instrumentator import Instrumentator

app = FastAPI()
Instrumentator().instrument(app).expose(app)
```

## 🔄 CI/CD Pipeline

### GitHub Actions

```yaml
# .github/workflows/deploy.yml
name: Deploy to Production

on:
  push:
    branches: [main]

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
        run: poetry run pytest tests/

  deploy:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Build and push Docker image
        uses: docker/build-push-action@v4
        with:
          push: true
          tags: registry.example.com/fastcar-api:latest
      
      - name: Deploy to server
        run: |
          ssh user@server "cd /app && docker-compose pull && docker-compose up -d"
```

## 📋 Rollback

### Reverter Deploy

```bash
# Docker Compose
docker-compose pull api:previous-version
docker-compose up -d api

# Kubernetes
kubectl rollout undo deployment/fastcar-api

# Heroku
heroku releases
heroku rollback v123
```

### Reverter Migração

```bash
# Reverter última migração
alembic downgrade -1

# Reverter para versão específica
alembic downgrade <revision_id>
```

## 🔍 Troubleshooting

### Problemas Comuns

**Erro: "Connection refused"**
```bash
# Verificar se banco está rodando
docker-compose ps

# Verificar logs
docker-compose logs db
```

**Erro: "Table doesn't exist"**
```bash
# Rodar migrações
docker-compose run --rm migrations
```

**Erro: "Permission denied"**
```bash
# Verificar permissões de arquivo
docker-compose exec api ls -la
```

## 📊 Comparação de Plataformas

| Plataforma | Custo | Facilidade | Escalabilidade |
|------------|-------|------------|----------------|
| Docker Compose | Baixo | Média | Baixa |
| AWS ECS | Médio | Média | Alta |
| Google Cloud Run | Médio | Alta | Alta |
| Heroku | Alto | Alta | Média |
| Railway | Médio | Alta | Média |

## 🚀 Próximo Passo

Com o deploy configurado, prossiga para [Contribuição](contributing.md).

---

**Dúvidas?** Consulte [Configuração](configuration.md) para detalhes de variáveis de ambiente.
