# Telegram UI Patterns

Паттерны пользовательского интерфейса для Telegram ботов.

---

## 🎯 Основные компоненты UI

### Типы взаимодействия

| Компонент | Когда использовать |
|-----------|-------------------|
| **Inline Keyboards** | Действия в контексте сообщения |
| **Reply Keyboards** | Постоянные опции (меню) |
| **Inline Mode** | Поиск и вставка контента |
| **Commands** | Быстрые действия (/start, /help) |

---

## ⌨️ Inline Keyboards

### Базовая структура

```python
from telegram import InlineKeyboardButton, InlineKeyboardMarkup

def build_menu_keyboard() -> InlineKeyboardMarkup:
    """Построение inline клавиатуры."""
    buttons = [
        [InlineKeyboardButton("📋 Мои задачи", callback_data="tasks_my")],
        [InlineKeyboardButton("➕ Создать", callback_data="task_create")],
        [
            InlineKeyboardButton("⚙️ Настройки", callback_data="settings"),
            InlineKeyboardButton("ℹ️ Помощь", callback_data="help"),
        ],
    ]
    return InlineKeyboardMarkup(buttons)
```

### Паттерны расположения кнопок

```
# Вертикальный список (одна кнопка в ряд)
┌─────────────────────────┐
│      📋 Мои задачи      │
├─────────────────────────┤
│      ➕ Создать         │
├─────────────────────────┤
│      ⚙️ Настройки       │
└─────────────────────────┘

# Горизонтальный ряд (несколько кнопок)
┌───────────┬───────────┐
│  ✅ Да    │  ❌ Нет   │
└───────────┴───────────┘

# Комбинированный
┌─────────────────────────┐
│      📋 Главное меню    │
├───────────┬─────────────┤
│  ◀️ Назад │  🔄 Обновить│
└───────────┴─────────────┘
```

### Динамическое построение

```python
def build_list_keyboard(
    items: list[Item],
    page: int = 1,
    per_page: int = 5
) -> InlineKeyboardMarkup:
    """Клавиатура со списком и пагинацией."""
    buttons = []
    
    # Элементы списка
    start = (page - 1) * per_page
    for item in items[start:start + per_page]:
        buttons.append([
            InlineKeyboardButton(
                f"{item.emoji} {item.title[:30]}",
                callback_data=f"item_{item.id}"
            )
        ])
    
    # Пагинация
    nav_row = []
    if page > 1:
        nav_row.append(InlineKeyboardButton("◀️", callback_data=f"page_{page-1}"))
    nav_row.append(InlineKeyboardButton(f"📄 {page}/{total_pages}", callback_data="noop"))
    if page < total_pages:
        nav_row.append(InlineKeyboardButton("▶️", callback_data=f"page_{page+1}"))
    
    if nav_row:
        buttons.append(nav_row)
    
    # Кнопка назад
    buttons.append([InlineKeyboardButton("🔙 Назад", callback_data="back")])
    
    return InlineKeyboardMarkup(buttons)
```

---

## 🧭 Навигация

### Breadcrumb паттерн

```python
def build_breadcrumb(path: list[str]) -> str:
    """Построение хлебных крошек."""
    return " → ".join(path)

# Примеры:
# "🏠 Главное меню"
# "🏠 Главное меню → 📋 Задачи"
# "🏠 Главное меню → 📋 Задачи → ✏️ Редактирование"
```

### Back-кнопка с состоянием

```python
# Хранение истории навигации
user_navigation_stack: dict[int, list[str]] = {}

def push_state(user_id: int, state: str):
    """Добавить состояние в стек."""
    if user_id not in user_navigation_stack:
        user_navigation_stack[user_id] = []
    user_navigation_stack[user_id].append(state)

def pop_state(user_id: int) -> str | None:
    """Вернуться к предыдущему состоянию."""
    stack = user_navigation_stack.get(user_id, [])
    return stack.pop() if stack else None
```

---

## 📝 Callback Data Conventions

### Структура callback_data

```python
# Паттерн: {action}_{entity}_{id}
# Максимум 64 байта!

# Примеры:
"task_view_123"      # Просмотр задачи 123
"task_edit_123"      # Редактирование задачи 123
"task_delete_123"    # Удаление задачи 123
"page_tasks_2"       # Страница 2 списка задач
"confirm_delete_123" # Подтверждение удаления
"cancel"             # Отмена действия
"back"               # Назад
"noop"               # Ничего не делать (для информационных кнопок)
```

### Парсинг callback_data

```python
async def handle_callback(update: Update, context: ContextTypes.DEFAULT_TYPE):
    """Обработка callback query."""
    query = update.callback_query
    await query.answer()  # Важно! Убирает "часики"
    
    data = query.data
    parts = data.split("_")
    
    if data == "back":
        await handle_back(query)
    elif data == "noop":
        return  # Ничего не делаем
    elif parts[0] == "task":
        action = parts[1]  # view, edit, delete
        task_id = int(parts[2])
        await handle_task_action(query, action, task_id)
    elif parts[0] == "page":
        entity = parts[1]
        page = int(parts[2])
        await handle_pagination(query, entity, page)
```

---

## 🎨 Emoji как UI элементы

### Статусы

| Emoji | Значение |
|-------|----------|
| ⏳ | Ожидание |
| 🔄 | В процессе |
| ✅ | Завершено |
| ❌ | Отклонено/Ошибка |
| 🚫 | Отменено |
| ⚠️ | Предупреждение |
| 🔴 | Критично/Просрочено |
| 🟡 | Требует внимания |
| 🟢 | Всё хорошо |

### Действия

| Emoji | Действие |
|-------|----------|
| ➕ | Создать |
| ✏️ | Редактировать |
| 🗑️ | Удалить |
| 👁️ | Просмотр |
| 🔙 | Назад |
| 🏠 | Главное меню |
| ⚙️ | Настройки |
| 📋 | Список |
| 🔍 | Поиск |

### Навигация

| Emoji | Значение |
|-------|----------|
| ◀️ | Предыдущая страница |
| ▶️ | Следующая страница |
| ⏮️ | В начало |
| ⏭️ | В конец |
| 🔄 | Обновить |

---

## 📊 Паттерны отображения данных

### Карточка элемента

```python
def format_task_card(task: Task) -> str:
    """Форматирование карточки задачи."""
    status_emoji = STATUS_EMOJI.get(task.status, "❓")
    
    lines = [
        f"{status_emoji} <b>{task.title}</b>",
        f"",
        f"📝 {task.description[:100]}{'...' if len(task.description) > 100 else ''}",
        f"",
        f"👤 Исполнитель: {task.assignee or 'Не назначен'}",
        f"📅 Срок: {task.due_date.strftime('%d.%m.%Y') if task.due_date else 'Не указан'}",
        f"📊 Статус: {STATUS_LABELS[task.status]}",
    ]
    
    return "\n".join(lines)
```

### Список элементов

```python
def format_task_list(tasks: list[Task], page: int, per_page: int) -> str:
    """Форматирование списка задач."""
    if not tasks:
        return "📭 Список пуст"
    
    lines = [f"📋 <b>Задачи</b> (стр. {page}):", ""]
    
    for i, task in enumerate(tasks, start=1):
        emoji = STATUS_EMOJI.get(task.status, "❓")
        overdue = "🔴 " if task.is_overdue else ""
        lines.append(f"{i}. {overdue}{emoji} {task.title[:40]}")
    
    lines.append("")
    lines.append(f"Всего: {len(tasks)}")
    
    return "\n".join(lines)
```

---

## ⚡ Best Practices

### Do's ✅

- Всегда вызывать `await query.answer()` для callback
- Использовать emoji для визуального различия
- Ограничивать callback_data до 64 байт
- Показывать breadcrumbs для навигации
- Давать возможность вернуться назад
- Использовать пагинацию для длинных списков

### Don'ts ❌

- Не делать слишком много кнопок (макс 8-10)
- Не использовать длинные тексты на кнопках
- Не забывать обрабатывать "noop" callbacks
- Не хранить чувствительные данные в callback_data
- Не делать глубокую вложенность меню (макс 3 уровня)

---

## 🔄 Обновление сообщений

```python
async def update_message(query: CallbackQuery, text: str, keyboard: InlineKeyboardMarkup):
    """Обновить сообщение с клавиатурой."""
    try:
        await query.edit_message_text(
            text=text,
            reply_markup=keyboard,
            parse_mode='HTML'
        )
    except Exception as e:
        # Сообщение не изменилось или удалено
        logger.warning(f"Failed to update message: {e}")
```

---

## 📚 Полезные ссылки

- [Telegram Bot API - Inline Keyboards](https://core.telegram.org/bots/api#inlinekeyboardmarkup)
- [python-telegram-bot Wiki](https://github.com/python-telegram-bot/python-telegram-bot/wiki)
