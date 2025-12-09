# Repository Pattern

Абстракция доступа к данным.

---

## 🎯 Основная идея

Repository — это **абстракция** между бизнес-логикой и хранилищем данных.

```
┌─────────────┐     ┌────────────┐     ┌──────────────┐
│  Service    │────►│ Repository │────►│   Database   │
│  (логика)   │     │ (interface)│     │   (storage)  │
└─────────────┘     └────────────┘     └──────────────┘
                           │
                           ▼
                    ┌────────────┐
                    │  In-Memory │  (для тестов)
                    └────────────┘
```

---

## 🏗️ Реализация

### 1. Интерфейс (абстракция)

```python
from abc import ABC, abstractmethod
from typing import Generic, TypeVar, Optional

T = TypeVar('T')

class Repository(ABC, Generic[T]):
    """Базовый интерфейс репозитория."""
    
    @abstractmethod
    async def get_by_id(self, entity_id: int) -> Optional[T]:
        """Получить сущность по ID."""
        pass
    
    @abstractmethod
    async def get_all(self) -> list[T]:
        """Получить все сущности."""
        pass
    
    @abstractmethod
    async def create(self, entity: T) -> T:
        """Создать сущность."""
        pass
    
    @abstractmethod
    async def update(self, entity: T) -> T:
        """Обновить сущность."""
        pass
    
    @abstractmethod
    async def delete(self, entity_id: int) -> bool:
        """Удалить сущность."""
        pass
```

### 2. Специфичный интерфейс

```python
from domain.entities.user import User

class UserRepository(Repository[User]):
    """Интерфейс репозитория пользователей."""
    
    @abstractmethod
    async def get_by_email(self, email: str) -> Optional[User]:
        """Найти пользователя по email."""
        pass
    
    @abstractmethod
    async def get_by_username(self, username: str) -> Optional[User]:
        """Найти пользователя по username."""
        pass
    
    @abstractmethod
    async def exists_by_email(self, email: str) -> bool:
        """Проверить существование email."""
        pass
```

### 3. Реализация (SQLAlchemy)

```python
from sqlalchemy import select
from sqlalchemy.ext.asyncio import AsyncSession

class SQLAlchemyUserRepository(UserRepository):
    """SQLAlchemy реализация UserRepository."""
    
    def __init__(self, session: AsyncSession):
        self.session = session
    
    async def get_by_id(self, user_id: int) -> Optional[User]:
        result = await self.session.execute(
            select(UserModel).where(UserModel.id == user_id)
        )
        model = result.scalar_one_or_none()
        return self._to_entity(model) if model else None
    
    async def get_by_email(self, email: str) -> Optional[User]:
        result = await self.session.execute(
            select(UserModel).where(UserModel.email == email)
        )
        model = result.scalar_one_or_none()
        return self._to_entity(model) if model else None
    
    async def create(self, user: User) -> User:
        model = self._to_model(user)
        self.session.add(model)
        await self.session.flush()
        return self._to_entity(model)
    
    async def update(self, user: User) -> User:
        model = await self.session.get(UserModel, user.id)
        if model:
            model.username = user.username
            model.email = user.email
            await self.session.flush()
        return user
    
    async def delete(self, user_id: int) -> bool:
        model = await self.session.get(UserModel, user_id)
        if model:
            await self.session.delete(model)
            return True
        return False
    
    def _to_entity(self, model: UserModel) -> User:
        return User(
            id=model.id,
            username=model.username,
            email=model.email
        )
    
    def _to_model(self, entity: User) -> UserModel:
        return UserModel(
            id=entity.id,
            username=entity.username,
            email=entity.email
        )
```

### 4. Реализация для тестов

```python
class InMemoryUserRepository(UserRepository):
    """In-memory реализация для тестов."""
    
    def __init__(self):
        self._users: dict[int, User] = {}
        self._next_id = 1
    
    async def get_by_id(self, user_id: int) -> Optional[User]:
        return self._users.get(user_id)
    
    async def get_by_email(self, email: str) -> Optional[User]:
        for user in self._users.values():
            if user.email == email:
                return user
        return None
    
    async def create(self, user: User) -> User:
        user.id = self._next_id
        self._next_id += 1
        self._users[user.id] = user
        return user
    
    async def update(self, user: User) -> User:
        self._users[user.id] = user
        return user
    
    async def delete(self, user_id: int) -> bool:
        if user_id in self._users:
            del self._users[user_id]
            return True
        return False
```

---

## 🧪 Тестирование

```python
import pytest

@pytest.fixture
def user_repo():
    """In-memory репозиторий для тестов."""
    return InMemoryUserRepository()

@pytest.mark.asyncio
async def test_create_user(user_repo):
    user = User(id=None, username="john", email="john@example.com")
    
    created = await user_repo.create(user)
    
    assert created.id is not None
    assert created.username == "john"

@pytest.mark.asyncio
async def test_get_by_email(user_repo):
    user = User(id=None, username="john", email="john@example.com")
    await user_repo.create(user)
    
    found = await user_repo.get_by_email("john@example.com")
    
    assert found is not None
    assert found.username == "john"
```

---

## ✅ Преимущества

| Преимущество | Описание |
|--------------|----------|
| **Тестируемость** | In-memory для тестов |
| **Гибкость** | Легко сменить хранилище |
| **Абстракция** | Бизнес-логика не знает о БД |
| **Single Responsibility** | Один класс = один источник данных |

---

## ⚠️ Когда НЕ использовать

- Очень простые CRUD приложения
- Когда ORM уже предоставляет достаточную абстракцию
- Прототипы и MVP

