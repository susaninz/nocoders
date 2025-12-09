# 📋 Шаблоны проектов

Готовые шаблоны для быстрого старта новых проектов.

---

## 🤖 Telegram Bot Template

Минималистичный шаблон для Telegram бота с Clean Architecture.

### Структура

```
telegram-bot-template/
├── app/
│   ├── __init__.py
│   ├── bot_app.py           # Entry point
│   ├── core/
│   │   ├── config.py        # Configuration
│   │   └── logger.py        # Logging setup
│   ├── adapters/
│   │   └── telegram_adapter.py
│   ├── flows/
│   │   └── main_flow.py
│   └── infrastructure/
│       ├── database.py
│       └── models.py
├── tests/
│   └── conftest.py
├── requirements.txt
├── .env.example
├── .gitignore
└── README.md
```

### Использование

```bash
# Скопировать шаблон
cp -r templates/telegram-bot-template my-new-bot
cd my-new-bot

# Настроить
cp .env.example .env
# Отредактировать .env

# Установить зависимости
pip install -r requirements.txt

# Запустить
python -m app.bot_app
```

---

## 🚀 FastAPI Service Template

*Coming soon...*

---

## 📝 CLI Tool Template

*Coming soon...*

---

## 💡 Как добавить свой шаблон

1. Создать директорию в `templates/`
2. Добавить все необходимые файлы
3. Создать README.md с инструкциями
4. Обновить этот файл

