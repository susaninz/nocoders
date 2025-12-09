# Docker

Контейнеризация приложений.

---

## 🐳 Базовые команды

### Образы

```bash
# Сборка
docker build -t my-app:latest .
docker build -t my-app:1.0.0 -f Dockerfile.prod .

# Список
docker images

# Удаление
docker rmi my-app:latest
docker image prune  # Удалить неиспользуемые
```

### Контейнеры

```bash
# Запуск
docker run -d --name my-app -p 8000:8000 my-app:latest
docker run -it --rm my-app:latest /bin/bash  # Интерактивно

# Управление
docker ps              # Запущенные
docker ps -a           # Все
docker stop my-app
docker start my-app
docker restart my-app
docker rm my-app

# Логи
docker logs my-app
docker logs -f my-app  # Follow
```

---

## 📄 Dockerfile

### Python приложение

```dockerfile
# Multi-stage build для минимального размера
FROM python:3.11-slim as builder

WORKDIR /app

# Зависимости
COPY requirements.txt .
RUN pip install --no-cache-dir --user -r requirements.txt

# Production образ
FROM python:3.11-slim

WORKDIR /app

# Копируем установленные пакеты
COPY --from=builder /root/.local /root/.local
ENV PATH=/root/.local/bin:$PATH

# Копируем код
COPY app/ ./app/

# Запуск
CMD ["python", "-m", "app.main"]
```

### Best Practices

```dockerfile
# ✅ Используй .dockerignore
# ✅ Минимизируй слои (объединяй RUN)
# ✅ Кэширование: сначала requirements, потом код
# ✅ Multi-stage builds
# ✅ Не запускай от root в production

# Пример оптимизированного RUN
RUN apt-get update && \
    apt-get install -y --no-install-recommends gcc && \
    pip install --no-cache-dir -r requirements.txt && \
    apt-get purge -y gcc && \
    apt-get autoremove -y && \
    rm -rf /var/lib/apt/lists/*
```

---

## 🐙 Docker Compose

### docker-compose.yml

```yaml
version: '3.8'

services:
  app:
    build: .
    ports:
      - "8000:8000"
    environment:
      - DATABASE_URL=postgresql://user:pass@db:5432/app
      - REDIS_URL=redis://redis:6379
    depends_on:
      - db
      - redis
    volumes:
      - ./app:/app/app  # Для разработки
    restart: unless-stopped

  db:
    image: postgres:15
    environment:
      POSTGRES_USER: user
      POSTGRES_PASSWORD: pass
      POSTGRES_DB: app
    volumes:
      - postgres_data:/var/lib/postgresql/data
    ports:
      - "5432:5432"

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"

volumes:
  postgres_data:
```

### Команды

```bash
# Запуск
docker-compose up -d

# Остановка
docker-compose down
docker-compose down -v  # С удалением volumes

# Логи
docker-compose logs -f app

# Пересборка
docker-compose build
docker-compose up -d --build

# Выполнение команды в контейнере
docker-compose exec app python manage.py migrate
docker-compose exec db psql -U user -d app
```

---

## 🔧 .dockerignore

```dockerignore
# Git
.git
.gitignore

# Python
__pycache__
*.py[cod]
*.egg-info
.eggs
dist
build
venv
.venv

# IDE
.idea
.vscode

# Environment
.env
.env.local
.env*.local

# Tests
tests
pytest.ini
.pytest_cache
.coverage
htmlcov

# Docs
docs
*.md
!README.md

# Docker
Dockerfile*
docker-compose*
.docker

# Misc
*.log
.DS_Store
```

---

## 🏭 Production Tips

### Health Check

```dockerfile
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD curl -f http://localhost:8000/health || exit 1
```

### Environment Variables

```yaml
# docker-compose.prod.yml
services:
  app:
    env_file:
      - .env.production
    environment:
      - NODE_ENV=production
```

### Logging

```yaml
services:
  app:
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
        max-file: "3"
```

---

## 📊 Полезные команды

```bash
# Очистка
docker system prune -a     # Всё неиспользуемое
docker volume prune        # Только volumes

# Статистика
docker stats               # Ресурсы контейнеров

# Inspect
docker inspect my-app
docker network ls
docker volume ls
```

