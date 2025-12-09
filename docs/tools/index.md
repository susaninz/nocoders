# 🛠️ Инструменты

Инструменты и утилиты для разработки.

---

## 📑 Содержание

| Раздел | Описание |
|--------|----------|
| [Git](git.md) | Команды, workflow, best practices |
| [Docker](docker.md) | Контейнеризация приложений |

---

## 🔧 Базовый набор инструментов

### Редактор / IDE

| Инструмент | Использование |
|------------|---------------|
| **Cursor** | AI-powered IDE (основной) |
| VS Code | Универсальный редактор |
| PyCharm | Python-специфичная IDE |

### Терминал

```bash
# Рекомендуемые алиасы для ~/.zshrc или ~/.bashrc

# Git
alias gs="git status"
alias gp="git pull"
alias gc="git commit"
alias gd="git diff"

# Python
alias py="python3"
alias venv="source venv/bin/activate"
alias req="pip install -r requirements.txt"

# Docker
alias dc="docker-compose"
alias dps="docker ps"
```

### Python инструменты

```bash
# Установка базового набора
pip install black isort mypy flake8 pytest pytest-asyncio
```

---

## 📦 Управление зависимостями

### Poetry (рекомендуется)

```bash
# Установка
curl -sSL https://install.python-poetry.org | python3 -

# Создание проекта
poetry new my-project

# Добавление зависимостей
poetry add sqlalchemy
poetry add --group dev pytest black

# Установка
poetry install

# Запуск
poetry run python main.py
```

### pip + requirements.txt

```bash
# Создание виртуального окружения
python -m venv venv
source venv/bin/activate

# Установка
pip install -r requirements.txt

# Сохранение зависимостей
pip freeze > requirements.txt
```

---

## 🐳 Docker Quick Reference

```bash
# Сборка образа
docker build -t my-app .

# Запуск контейнера
docker run -d --name my-app -p 8000:8000 my-app

# Docker Compose
docker-compose up -d
docker-compose logs -f
docker-compose down
```

