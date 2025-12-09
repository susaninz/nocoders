# SQLAlchemy Best Practices

Работа с SQLAlchemy 2.0 в async режиме.

---

## 🔌 Connection Pooling

!!! warning "Критически важно"
    Connection pooling — обязательная практика для любого приложения с БД!

### Настройка пула

```python
from sqlalchemy.ext.asyncio import create_async_engine, AsyncSession
from sqlalchemy.orm import sessionmaker

engine = create_async_engine(
    "sqlite+aiosqlite:///app.db",
    pool_size=5,          # Базовое количество соединений
    max_overflow=10,      # Дополнительные при пиках
    pool_pre_ping=True,   # Проверка живости
    pool_recycle=3600,    # Обновление каждый час
    pool_timeout=30,      # Таймаут ожидания
)

async_session_factory = sessionmaker(
    engine,
    class_=AsyncSession,
    expire_on_commit=False
)
```

### Параметры по нагрузке

| Нагрузка | pool_size | max_overflow |
|----------|-----------|--------------|
| Низкая (<100 users) | 3-5 | 5 |
| Средняя (100-1000) | 5-10 | 10-15 |
| Высокая (1000+) | 10-20 | 20-30 |

---

## 📊 Модели

### Базовая модель

```python
from sqlalchemy import Column, Integer, String, DateTime, ForeignKey
from sqlalchemy.orm import relationship, declarative_base
from sqlalchemy.sql import func

Base = declarative_base()

class User(Base):
    __tablename__ = "users"
    
    id = Column(Integer, primary_key=True, index=True)
    username = Column(String(255), unique=True, nullable=False)
    email = Column(String(255), nullable=False)
    created_at = Column(DateTime(timezone=True), server_default=func.now())
    updated_at = Column(DateTime(timezone=True), onupdate=func.now())
    
    # Relationships
    tasks = relationship("Task", back_populates="creator")
```

---

## 🔍 Запросы

### Базовые операции

```python
from sqlalchemy import select
from sqlalchemy.orm import selectinload

async def get_user(session: AsyncSession, user_id: int) -> User | None:
    result = await session.execute(
        select(User).where(User.id == user_id)
    )
    return result.scalar_one_or_none()

async def get_users_with_tasks(session: AsyncSession) -> list[User]:
    """Eager loading для избежания N+1."""
    result = await session.execute(
        select(User).options(selectinload(User.tasks))
    )
    return list(result.scalars().all())
```

### Фильтрация

```python
from sqlalchemy import and_, or_

async def search_users(
    session: AsyncSession,
    query: str,
    is_active: bool = True
) -> list[User]:
    result = await session.execute(
        select(User).where(
            and_(
                User.is_active == is_active,
                or_(
                    User.username.ilike(f"%{query}%"),
                    User.email.ilike(f"%{query}%")
                )
            )
        )
    )
    return list(result.scalars().all())
```

---

## ⚠️ Частые ошибки

### N+1 проблема

```python
# ❌ ПЛОХО: N+1 запросов
users = await session.execute(select(User))
for user in users.scalars():
    print(user.tasks)  # Каждый раз новый запрос!

# ✅ ХОРОШО: Один запрос с eager loading
users = await session.execute(
    select(User).options(selectinload(User.tasks))
)
```

### Долгоживущие сессии

```python
# ❌ ПЛОХО: Одна сессия на всё приложение
session = AsyncSession(engine)  # Создаётся при старте

# ✅ ХОРОШО: Сессия на запрос
async with async_session_factory() as session:
    # Работаем с БД
    await session.commit()
# Сессия автоматически закрывается
```

---

## 📚 Дополнительно

- [SQLAlchemy 2.0 Docs](https://docs.sqlalchemy.org/en/20/)
- [Async SQLAlchemy](https://docs.sqlalchemy.org/en/20/orm/extensions/asyncio.html)

