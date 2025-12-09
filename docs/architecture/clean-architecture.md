# Clean Architecture

Чистая архитектура и разделение слоёв.

---

## 🎯 Основная идея

> **Зависимости направлены внутрь** — внутренние слои ничего не знают о внешних.

```
┌─────────────────────────────────────────────────────────────┐
│                    FRAMEWORKS & DRIVERS                      │
│                   (Web, DB, External APIs)                   │
│  ┌───────────────────────────────────────────────────────┐  │
│  │                 INTERFACE ADAPTERS                     │  │
│  │              (Controllers, Gateways, Presenters)       │  │
│  │  ┌─────────────────────────────────────────────────┐  │  │
│  │  │              APPLICATION BUSINESS                │  │  │
│  │  │                 (Use Cases)                      │  │  │
│  │  │  ┌───────────────────────────────────────────┐  │  │  │
│  │  │  │         ENTERPRISE BUSINESS               │  │  │  │
│  │  │  │            (Entities)                     │  │  │  │
│  │  │  └───────────────────────────────────────────┘  │  │  │
│  │  └─────────────────────────────────────────────────┘  │  │
│  └───────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘

         Направление зависимостей: ────────────────►
                                   (снаружи внутрь)
```

---

## 📁 Структура проекта

```
app/
├── domain/              # Enterprise Business (центр)
│   ├── entities/        # Бизнес-объекты
│   ├── value_objects/   # Неизменяемые объекты
│   └── events/          # Доменные события
│
├── application/         # Application Business
│   ├── use_cases/       # Сценарии использования
│   ├── services/        # Сервисы приложения
│   └── interfaces/      # Абстракции (порты)
│
├── infrastructure/      # Frameworks & Drivers
│   ├── database/        # Реализация persistence
│   ├── external/        # Внешние API
│   └── config/          # Конфигурация
│
└── presentation/        # Interface Adapters
    ├── api/             # REST/GraphQL controllers
    ├── cli/             # CLI handlers
    └── telegram/        # Bot handlers
```

---

## 🧱 Слои и их ответственности

### 1. Domain Layer (Entities)

**Что**: Бизнес-объекты с правилами  
**Зависимости**: Никаких!

```python
# domain/entities/user.py
from dataclasses import dataclass
from enum import Enum

class UserRole(Enum):
    ADMIN = "admin"
    USER = "user"

@dataclass
class User:
    id: int
    username: str
    email: str
    role: UserRole = UserRole.USER
    
    def can_manage_users(self) -> bool:
        """Бизнес-правило в сущности."""
        return self.role == UserRole.ADMIN
```

### 2. Application Layer (Use Cases)

**Что**: Бизнес-логика приложения  
**Зависимости**: Только от Domain

```python
# application/use_cases/create_user.py
from domain.entities.user import User
from application.interfaces.user_repository import UserRepository

class CreateUserUseCase:
    def __init__(self, user_repo: UserRepository):
        self.user_repo = user_repo
    
    async def execute(self, username: str, email: str) -> User:
        # Бизнес-логика
        if await self.user_repo.exists_by_email(email):
            raise UserAlreadyExistsError(email)
        
        user = User(id=None, username=username, email=email)
        return await self.user_repo.save(user)
```

### 3. Infrastructure Layer

**Что**: Реализация интерфейсов  
**Зависимости**: От Application (реализует интерфейсы)

```python
# infrastructure/database/user_repository_impl.py
from application.interfaces.user_repository import UserRepository
from domain.entities.user import User

class SQLAlchemyUserRepository(UserRepository):
    def __init__(self, session: AsyncSession):
        self.session = session
    
    async def save(self, user: User) -> User:
        model = UserModel(**user.__dict__)
        self.session.add(model)
        await self.session.commit()
        return user
```

### 4. Presentation Layer

**Что**: Обработка входящих запросов  
**Зависимости**: От Application (вызывает use cases)

```python
# presentation/api/user_controller.py
from application.use_cases.create_user import CreateUserUseCase

class UserController:
    def __init__(self, create_user: CreateUserUseCase):
        self.create_user = create_user
    
    async def post(self, request: CreateUserRequest) -> UserResponse:
        user = await self.create_user.execute(
            username=request.username,
            email=request.email
        )
        return UserResponse.from_entity(user)
```

---

## 🔄 Dependency Inversion

### Интерфейсы (порты)

```python
# application/interfaces/user_repository.py
from abc import ABC, abstractmethod
from domain.entities.user import User

class UserRepository(ABC):
    @abstractmethod
    async def save(self, user: User) -> User:
        pass
    
    @abstractmethod
    async def get_by_id(self, user_id: int) -> User | None:
        pass
```

### Инъекция зависимостей

```python
# Composition Root
def create_app():
    # Infrastructure
    engine = create_async_engine(DATABASE_URL)
    session_factory = sessionmaker(engine, class_=AsyncSession)
    
    # Repositories
    user_repo = SQLAlchemyUserRepository(session_factory)
    
    # Use Cases
    create_user = CreateUserUseCase(user_repo)
    
    # Controllers
    user_controller = UserController(create_user)
    
    return App(user_controller=user_controller)
```

---

## ✅ Преимущества

| Преимущество | Описание |
|--------------|----------|
| **Тестируемость** | Легко мокать зависимости |
| **Гибкость** | Легко менять реализации |
| **Независимость** | Domain не зависит от фреймворков |
| **Понятность** | Чёткие границы ответственности |

---

## ⚠️ Когда НЕ использовать

- Очень простые проекты (overcomplicated)
- Прототипы и MVP
- Скрипты и утилиты
- Когда скорость разработки важнее архитектуры

