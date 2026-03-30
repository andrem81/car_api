# Pré-requisitos

Antes de começar a trabalhar com a Fastcar API, certifique-se de ter os seguintes requisitos instalados e configurados em seu ambiente.

## 🖥️ Sistema Operacional

A API é compatível com os principais sistemas operacionais:

- ✅ **Linux** (Ubuntu, Debian, Fedora, CentOS, etc.)
- ✅ **macOS** (10.15 ou superior)
- ✅ **Windows** (10 ou superior)

## 🐍 Python

### Versão Requerida

- **Python 3.13** ou superior (compatível com `<4.0`)

### Verificação da Instalação

```bash
python --version
# ou
python3 --version
```

### Instalação do Python

#### Ubuntu/Debian

```bash
sudo apt update
sudo apt install python3.13 python3.13-venv python3.13-dev
```

#### macOS (com Homebrew)

```bash
brew install python@3.13
```

#### Windows

Baixe o instalador em [python.org](https://www.python.org/downloads/)

## 📦 Poetry

O **Poetry** é o gerenciador de dependências e empacotamento utilizado no projeto.

### Versão Requerida

- **Poetry 2.0.0** ou superior

### Verificação da Instalação

```bash
poetry --version
```

### Instalação do Poetry

#### Via pipx (Recomendado)

O uso do `pipx` é recomendado para isolar o Poetry em um ambiente virtual dedicado:

```bash
pipx install poetry
```

> **Nota**: Se não tiver o pipx instalado:
> ```bash
> pip install pipx
> pipx ensurepath
> ```

#### Método Oficial

```bash
curl -sSL https://install.python-poetry.org | python3 -
```

#### Via pip

```bash
pip install poetry
```

#### Via Homebrew (macOS)

```bash
brew install poetry
```

### Configuração do Path

Após a instalação, adicione o Poetry ao seu PATH:

```bash
# Linux/macOS
export PATH="$HOME/.local/bin:$PATH"

# Ou adicione ao ~/.bashrc ou ~/.zshrc
echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
```

## 🗄️ Banco de Dados

### SQLite

O projeto utiliza **SQLite** como banco de dados padrão (via `aiosqlite` para operações assíncronas).

- ✅ Já incluído na biblioteca padrão do Python
- ✅ Não requer instalação adicional
- ✅ Ideal para desenvolvimento e testes

### Bancos Suportados (Produção)

Para ambientes de produção, a API suporta:

- **PostgreSQL** (recomendado)
- **MySQL**
- **MariaDB**

> **Nota**: Para usar outros bancos, ajuste a `DATABASE_URL` no arquivo `.env` e instale o driver apropriado.

## 🛠️ Ferramentas de Desenvolvimento

### Git

Controle de versão é essencial para contribuir com o projeto.

```bash
git --version
```

**Instalação**:

```bash
# Ubuntu/Debian
sudo apt install git

# macOS
brew install git

# Windows
# Baixe em https://git-scm.com/download/win
```

### Editor de Código / IDE

Recomendações:

- **VS Code** (com extensões Python, Pylance, Ruff)
- **PyCharm** (Professional ou Community)
- **Vim/Neovim** (com configuração Python)

### Extensões Recomendadas (VS Code)

- Python (Microsoft)
- Pylance
- Ruff
- Docker (opcional)
- GitLens (opcional)

## 🌐 Conhecimentos Necessários

Para trabalhar efetivamente com este projeto, é recomendável ter conhecimento em:

| Tecnologia | Nível | Descrição |
|------------|-------|-----------|
| Python | Intermediário | Sintaxe, type hints, async/await |
| FastAPI | Básico | Routers, Depends, status codes |
| SQLAlchemy | Intermediário | ORM, models, relationships |
| Pydantic | Básico | Schemas, validação de dados |
| JWT | Básico | Autenticação, tokens |
| Git | Intermediário | Branches, commits, pull requests |
| REST API | Intermediário | Verbos HTTP, status codes |

## 📋 Checklist de Verificação

Antes de prosseguir, verifique:

```bash
# Python 3.13+ instalado
python --version  # Deve mostrar Python 3.13.x

# Poetry instalado
poetry --version  # Deve mostrar Poetry 2.x.x

# Git instalado
git --version  # Deve mostrar git version 2.x.x

# Editor de código configurado
# (Verifique manualmente)
```

## 🚀 Próximo Passo

Com todos os pré-requisitos instalados, prossiga para o guia de [Instalação](installation.md).

---

**Dúvidas?** Consulte a seção de [Desenvolvimento](development.md) para mais informações sobre o ambiente de desenvolvimento.
