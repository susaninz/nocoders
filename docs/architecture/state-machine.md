# State Machine

Конечные автоматы для управления состоянием.

---

## 🎯 Когда использовать

State Machine идеален когда:

- Объект имеет **конечное число состояний**
- Есть **чёткие переходы** между состояниями
- Переходы зависят от **событий/действий**
- Нужна **валидация** допустимых переходов

**Примеры**: Заказы, задачи, документы, платежи, подписки.

---

## 📊 Пример: Жизненный цикл задачи

```
                    ┌──────────┐
                    │  DRAFT   │
                    └────┬─────┘
                         │ assign
                         ▼
               ┌───────────────────┐
        ┌──────│    AWAITING       │──────┐
        │      └────────┬──────────┘      │
        │ reject        │ accept          │ cancel
        ▼               ▼                 ▼
┌──────────┐      ┌──────────┐      ┌──────────┐
│ REJECTED │      │ ACCEPTED │      │ CANCELLED│
└──────────┘      └────┬─────┘      └──────────┘
                       │ complete
                       ▼
                 ┌──────────┐
                 │ COMPLETED│
                 └──────────┘
```

---

## 🏗️ Реализация

### 1. Определение состояний и событий

```python
from enum import Enum

class TaskState(Enum):
    DRAFT = "draft"
    AWAITING = "awaiting"
    ACCEPTED = "accepted"
    COMPLETED = "completed"
    REJECTED = "rejected"
    CANCELLED = "cancelled"

class TaskEvent(Enum):
    ASSIGN = "assign"
    ACCEPT = "accept"
    REJECT = "reject"
    COMPLETE = "complete"
    CANCEL = "cancel"
```

### 2. Таблица переходов

```python
from dataclasses import dataclass
from typing import Callable, Optional

@dataclass
class Transition:
    from_state: TaskState
    to_state: TaskState
    event: TaskEvent
    guard: Optional[Callable] = None  # Условие перехода
    action: Optional[Callable] = None  # Действие при переходе

TRANSITIONS = [
    Transition(TaskState.DRAFT, TaskState.AWAITING, TaskEvent.ASSIGN),
    Transition(TaskState.AWAITING, TaskState.ACCEPTED, TaskEvent.ACCEPT,
               guard=lambda ctx: ctx['user_id'] == ctx['executor_id']),
    Transition(TaskState.AWAITING, TaskState.REJECTED, TaskEvent.REJECT),
    Transition(TaskState.ACCEPTED, TaskState.COMPLETED, TaskEvent.COMPLETE),
    Transition(TaskState.AWAITING, TaskState.CANCELLED, TaskEvent.CANCEL),
    Transition(TaskState.ACCEPTED, TaskState.CANCELLED, TaskEvent.CANCEL),
]
```

### 3. State Machine класс

```python
class StateMachine:
    def __init__(self, initial_state: TaskState):
        self.state = initial_state
        self.transitions = {
            (t.from_state, t.event): t for t in TRANSITIONS
        }
    
    def can_trigger(self, event: TaskEvent, context: dict = None) -> bool:
        """Проверить, можно ли выполнить переход."""
        key = (self.state, event)
        transition = self.transitions.get(key)
        
        if not transition:
            return False
        
        if transition.guard and context:
            return transition.guard(context)
        
        return True
    
    async def trigger(self, event: TaskEvent, context: dict = None) -> TaskState:
        """Выполнить переход."""
        if not self.can_trigger(event, context):
            raise InvalidTransitionError(
                f"Cannot {event.value} from {self.state.value}"
            )
        
        transition = self.transitions[(self.state, event)]
        
        # Выполнить action если есть
        if transition.action:
            await transition.action(context)
        
        self.state = transition.to_state
        return self.state
    
    def get_available_events(self) -> list[TaskEvent]:
        """Получить доступные события из текущего состояния."""
        return [
            event for (state, event) in self.transitions.keys()
            if state == self.state
        ]
```

### 4. Использование

```python
# Создание
task_sm = StateMachine(TaskState.DRAFT)

# Проверка доступных действий
available = task_sm.get_available_events()
# [TaskEvent.ASSIGN]

# Выполнение перехода
await task_sm.trigger(TaskEvent.ASSIGN)
# state = TaskState.AWAITING

# С контекстом (для guard)
await task_sm.trigger(
    TaskEvent.ACCEPT,
    context={'user_id': 1, 'executor_id': 1}
)
# state = TaskState.ACCEPTED

# Невалидный переход
await task_sm.trigger(TaskEvent.ASSIGN)
# raises InvalidTransitionError
```

---

## 🎨 UI интеграция

### Динамические кнопки

```python
def build_task_keyboard(task: Task, user_id: int) -> InlineKeyboard:
    """Строим клавиатуру на основе доступных переходов."""
    sm = StateMachine(TaskState(task.status))
    context = {
        'user_id': user_id,
        'creator_id': task.creator_id,
        'executor_id': task.executor_id,
    }
    
    buttons = []
    for event in sm.get_available_events():
        if sm.can_trigger(event, context):
            buttons.append(
                Button(text=EVENT_LABELS[event], callback=f"{event.value}_{task.id}")
            )
    
    return InlineKeyboard(buttons)
```

---

## ✅ Преимущества

| Преимущество | Описание |
|--------------|----------|
| **Предсказуемость** | Только валидные переходы |
| **Документация** | Диаграмма = код |
| **Тестируемость** | Легко тестировать переходы |
| **Расширяемость** | Добавление состояний/событий |
| **Аудит** | История переходов |

---

## 📚 Библиотеки

- [transitions](https://github.com/pytransitions/transitions) — популярная Python библиотека
- [python-statemachine](https://github.com/fgmacedo/python-statemachine) — декларативный подход

