# 🤖 Telegram боты

Разработка Telegram ботов на Python.

---

## 📑 Содержание

| Раздел | Описание |
|--------|----------|
| [Подход к разработке](development-approach.md) | Минималистичный workflow-centric подход |
| [UI паттерны](ui-patterns.md) | Inline keyboards, навигация, callback data |

---

## 🛠️ Технологический стек

| Компонент | Рекомендация | Альтернативы |
|-----------|--------------|--------------|
| Библиотека | python-telegram-bot | aiogram, Telethon |
| База данных | SQLAlchemy + aiosqlite/asyncpg | — |
| Валидация | Pydantic | dataclasses |
| Конфигурация | python-dotenv | pydantic-settings |

---

## 🏗️ Типовая структура проекта

```
bot-project/
├── app/
│   ├── adapters/          # Telegram API adapter
│   ├── core/              # Config, logging
│   ├── domain/            # Entities, state machine
│   ├── flows/             # User workflows
│   ├── infrastructure/    # Database, models
│   ├── ops/               # Business operations
│   ├── repositories/      # Data access
│   ├── services/          # Business services
│   └── ui/                # Keyboards, templates
├── tests/
├── requirements.txt
├── .env.example
└── README.md
```

---

## 🎯 Ключевые паттерны

### Workflow-Centric Design

Организуй код вокруг **пользовательских потоков**, а не отдельных функций:

```python
class TaskCreationFlow:
    """Полный цикл создания задачи."""
    
    async def start(self, message):
        # 1. Парсинг сообщения
        # 2. Валидация
        # 3. Создание в БД
        # 4. Уведомления
        # 5. UI feedback
```

### State Machine для статусов

Используй конечный автомат для управления состояниями:

```python
class TaskState(Enum):
    DRAFT = "draft"
    AWAITING = "awaiting_acceptance"
    ACCEPTED = "accepted"
    COMPLETED = "completed"
    CANCELLED = "cancelled"
```

### Emoji как индикаторы статуса

| Статус | Emoji |
|--------|-------|
| Ожидает | 🟡 |
| В работе | 🟢 |
| Завершено | ✅ |
| Отклонено | ❌ |
| Отменено | 🚫 |

---

## ⚡ Quick Start

```bash
# 1. Создать бота через @BotFather и получить токен

# 2. Создать .env файл
echo "TELEGRAM_BOT_TOKEN=your_token_here" > .env

# 3. Установить зависимости
pip install python-telegram-bot sqlalchemy aiosqlite python-dotenv

# 4. Запустить
python -m app.bot_app
```

---

## 📚 Связанные разделы

- [Clean Architecture](../architecture/clean-architecture.md) — архитектура приложения
- [State Machine](../architecture/state-machine.md) — управление состоянием
- [Repository Pattern](../architecture/repository-pattern.md) — доступ к данным
- [Dependency Lifecycle](../architecture/dependency-lifecycle.md) — управление зависимостями
- [SQLAlchemy](../python/sqlalchemy.md) — работа с БД
- [Async паттерны](../python/async-patterns.md) — асинхронный код
