# Development Practices

Universal development practices for modern Python projects with async-first architecture, comprehensive testing, and clean code principles.

## 📋 Table of Contents

1. [Development Environment Setup](#development-environment-setup)
2. [Project Structure](#project-structure)
3. [Dependency Management](#dependency-management)
4. [Code Standards](#code-standards)
5. [Development Workflow](#development-workflow)
6. [Tool Configuration](#tool-configuration)
7. [Best Practices](#best-practices)

## 🛠️ Development Environment Setup

### Prerequisites

- **Python 3.11+** - Modern async features and performance
- **Poetry** - Modern dependency management
- **Git** - Version control
- **Docker** (optional) - Containerization

### Quick Setup

```bash
# Install Poetry
curl -sSL https://install.python-poetry.org | python3 -

# Create new project
poetry new my-project
cd my-project

# Install dependencies
poetry install

# Activate virtual environment
poetry shell
```

### IDE Configuration

#### VS Code / Cursor Setup

1. **Install Extensions**:

   ```json
   {
     "extensions": [
       "ms-python.python",
       "ms-python.black-formatter",
       "ms-python.flake8",
       "ms-python.mypy-type-checker",
       "ms-python.pytest-adapter"
     ]
   }
   ```

2. **Create `.vscode/settings.json`**:

   ```json
   {
     "python.defaultInterpreterPath": "./venv/bin/python",
     "python.formatting.provider": "black",
     "python.linting.enabled": true,
     "python.linting.flake8Enabled": true,
     "python.linting.mypyEnabled": true,
     "python.testing.pytestEnabled": true,
     "python.testing.pytestArgs": ["tests/"],
     "editor.formatOnSave": true,
     "editor.codeActionsOnSave": {
       "source.organizeImports": true
     }
   }
   ```

## 📁 Project Structure

### Recommended Structure

```
project/
├── src/                          # Source code
│   ├── core/                     # Business logic (framework-agnostic)
│   │   ├── models/               # Domain models
│   │   ├── services/             # Business services
│   │   ├── repositories/         # Data access interfaces
│   │   └── exceptions/           # Custom exceptions
│   ├── infrastructure/           # External concerns
│   │   ├── database/             # Database implementations
│   │   ├── external/             # External API clients
│   │   └── config/               # Configuration
│   └── api/                      # Interface layer
│       ├── web/                  # Web API (FastAPI/Flask)
│       ├── cli/                  # Command line interface
│       └── telegram/             # Telegram bot (if applicable)
├── tests/                        # Test files
│   ├── unit/                     # Unit tests
│   ├── integration/              # Integration tests
│   └── conftest.py              # Test configuration
├── docs/                         # Documentation
├── scripts/                      # Utility scripts
├── pyproject.toml               # Poetry configuration
├── pytest.ini                  # Test configuration
├── .env.example                 # Environment variables template
└── README.md                    # Project documentation
```

### Layer Responsibilities

#### Core Layer (`src/core/`)

- **Pure business logic** - No external dependencies
- **Domain models** - Business entities and rules
- **Services** - Business operations and workflows
- **Repository interfaces** - Data access contracts

#### Infrastructure Layer (`src/infrastructure/`)

- **Database implementations** - SQLAlchemy models and repositories
- **External integrations** - API clients, file systems
- **Configuration** - Environment and settings management

#### API Layer (`src/api/`)

- **Interface implementations** - Web, CLI, or bot interfaces
- **Request/response handling** - Input validation and formatting
- **Authentication** - User authentication and authorization

## 📦 Dependency Management

### Poetry Configuration

#### `pyproject.toml` Structure

```toml
[tool.poetry]
name = "my-project"
version = "0.1.0"
description = "Project description"
authors = ["Your Name <your.email@example.com>"]
readme = "README.md"

[tool.poetry.dependencies]
python = "^3.11"
sqlalchemy = "^2.0"
alembic = "^1.12"
aiosqlite = "^0.19"
pydantic = "^2.0"
python-dotenv = "^1.0"

[tool.poetry.group.dev.dependencies]
pytest = "^7.4"
pytest-asyncio = "^0.21"
mypy = "^1.5"
black = "^23.0"
flake8 = "^6.0"
isort = "^5.12"

[tool.poetry.scripts]
my-app = "src.main:main"

[build-system]
requires = ["poetry-core"]
build-backend = "poetry.core.masonry.api"

[tool.black]
line-length = 88
target-version = ['py311']

[tool.mypy]
python_version = "3.11"
strict = true
warn_return_any = true
warn_unused_configs = true

[tool.pytest.ini_options]
asyncio_mode = "auto"
testpaths = ["tests"]
python_files = ["test_*.py"]
python_classes = ["Test*"]
python_functions = ["test_*"]
```

### Dependency Groups

```bash
# Install all dependencies
poetry install

# Install only production dependencies
poetry install --only=main

# Install with development dependencies
poetry install --with dev

# Add new dependency
poetry add package-name

# Add development dependency
poetry add --group dev package-name

# Add test dependency
poetry add --group test package-name
```

## 📝 Code Standards

### Type Hints

```python
from typing import Optional, List, Dict, Any, Union
from dataclasses import dataclass
from enum import Enum

@dataclass
class User:
    id: Optional[int]
    username: str
    email: str
    is_active: bool = True
    
    def get_display_name(self) -> str:
        return self.username.title()

class UserRole(Enum):
    ADMIN = "admin"
    USER = "user"
    GUEST = "guest"

async def get_user_by_id(user_id: int) -> Optional[User]:
    """Get user by ID with proper type hints."""
    # Implementation here
    pass
```

### Async/Await Patterns

```python
import asyncio
from typing import List

async def process_users(user_ids: List[int]) -> List[User]:
    """Process multiple users concurrently."""
    tasks = [get_user_by_id(user_id) for user_id in user_ids]
    users = await asyncio.gather(*tasks, return_exceptions=True)
    
    # Filter out exceptions
    return [user for user in users if isinstance(user, User)]

async def create_user_with_validation(user_data: Dict[str, Any]) -> User:
    """Create user with async validation."""
    # Validate input
    if not user_data.get('username'):
        raise ValueError("Username is required")
    
    # Create user
    user = User(
        id=None,
        username=user_data['username'],
        email=user_data['email']
    )
    
    # Save to database
    user_id = await user_repository.create(user)
    user.id = user_id
    
    return user
```

### Error Handling

```python
class ServiceError(Exception):
    """Base exception for service errors."""
    pass

class UserNotFoundError(ServiceError):
    """Raised when user is not found."""
    pass

class ValidationError(ServiceError):
    """Raised when validation fails."""
    pass

async def get_user_safely(user_id: int) -> User:
    """Get user with proper error handling."""
    try:
        user = await user_repository.get_by_id(user_id)
        if not user:
            raise UserNotFoundError(f"User {user_id} not found")
        return user
    except DatabaseError as e:
        raise ServiceError(f"Database error: {e}") from e
```

## 🔄 Development Workflow

### 1. Feature Development Process

```bash
# 1. Create feature branch
git checkout -b feature/user-authentication

# 2. Write tests first (TDD)
# Write test in tests/unit/test_user_service.py

# 3. Implement feature
# Implement in src/core/services/user_service.py

# 4. Run tests
poetry run pytest tests/unit/test_user_service.py -v

# 5. Run all tests
poetry run pytest

# 6. Format and lint
poetry run black .
poetry run flake8 src/
poetry run mypy src/

# 7. Commit changes
git add .
git commit -m "feat: add user authentication service"

# 8. Push and create PR
git push origin feature/user-authentication
```

### 2. Test-Driven Development

```python
# 1. Write failing test
@pytest.mark.asyncio
async def test_create_user_with_valid_data():
    """Test creating user with valid data."""
    user_data = {
        "username": "testuser",
        "email": "test@example.com"
    }
    
    user = await user_service.create_user(user_data)
    
    assert user.username == "testuser"
    assert user.email == "test@example.com"
    assert user.is_active is True

# 2. Run test (should fail)
poetry run pytest tests/unit/test_user_service.py::test_create_user_with_valid_data -v

# 3. Implement minimal code to pass
async def create_user(self, user_data: Dict[str, Any]) -> User:
    user = User(
        id=None,
        username=user_data["username"],
        email=user_data["email"]
    )
    return user

# 4. Run test (should pass)
poetry run pytest tests/unit/test_user_service.py::test_create_user_with_valid_data -v

# 5. Refactor and add more tests
```

### 3. Code Review Checklist

- [ ] **Tests written and passing**
- [ ] **Type hints complete**
- [ ] **Error handling implemented**
- [ ] **Documentation updated**
- [ ] **Code formatted with Black**
- [ ] **Linting passes**
- [ ] **No hardcoded values**
- [ ] **Async/await used correctly**
- [ ] **Repository pattern followed**
- [ ] **Service layer used for business logic**

## ⚙️ Tool Configuration

### Black (Code Formatting)

```toml
# pyproject.toml
[tool.black]
line-length = 88
target-version = ['py311']
include = '\.pyi?$'
extend-exclude = '''
/(
  # directories
  \.eggs
  | \.git
  | \.hg
  | \.mypy_cache
  | \.tox
  | \.venv
  | build
  | dist
)/
'''
```

### MyPy (Type Checking)

```toml
# pyproject.toml
[tool.mypy]
python_version = "3.11"
strict = true
warn_return_any = true
warn_unused_configs = true
disallow_untyped_defs = true
disallow_incomplete_defs = true
check_untyped_defs = true
disallow_untyped_decorators = true
no_implicit_optional = true
warn_redundant_casts = true
warn_unused_ignores = true
warn_no_return = true
warn_unreachable = true
strict_equality = true

[[tool.mypy.overrides]]
module = "tests.*"
disallow_untyped_defs = false
```

### Pytest (Testing)

```ini
# pytest.ini
[tool:pytest]
asyncio_mode = auto
testpaths = tests
python_files = test_*.py
python_classes = Test*
python_functions = test_*
addopts = 
    -v
    --tb=short
    --strict-markers
    --disable-warnings
    --cov=src
    --cov-report=term-missing
    --cov-report=html
markers =
    unit: Unit tests
    integration: Integration tests
    slow: Slow tests
```

### Flake8 (Linting)

```ini
# setup.cfg
[flake8]
max-line-length = 88
extend-ignore = E203, W503
exclude = 
    .git,
    __pycache__,
    .venv,
    venv,
    .eggs,
    *.egg,
    build,
    dist
```

## 🎯 Best Practices

### 1. Async Development

- **Always use async/await** for I/O operations
- **Use asyncio.gather()** for concurrent operations
- **Handle exceptions properly** in async functions
- **Use async context managers** for resource management

### 2. Connection Pooling (Database)

Connection pooling — **обязательная практика** для любого приложения с базой данных. Пул поддерживает готовые соединения, избегая overhead на создание/закрытие при каждом запросе.

#### Проблема без пула

```python
# ❌ ПЛОХО: Новое соединение на каждый запрос
async def get_user(user_id: int):
    async with AsyncSession(engine) as session:  # ~50-100ms overhead
        return await session.get(User, user_id)
```

#### Решение с пулом

```python
from sqlalchemy.ext.asyncio import create_async_engine, AsyncSession
from sqlalchemy.orm import sessionmaker

# ✅ ХОРОШО: Настройка пула при инициализации
engine = create_async_engine(
    database_url,
    pool_size=5,          # Базовое количество соединений
    max_overflow=10,      # Дополнительные соединения при пиках
    pool_pre_ping=True,   # Проверка живости перед использованием
    pool_recycle=3600,    # Пересоздание соединений каждый час
    pool_timeout=30,      # Таймаут ожидания свободного соединения
    echo=False,           # Логирование SQL (True для отладки)
)

# Фабрика сессий
async_session_factory = sessionmaker(
    engine, 
    class_=AsyncSession, 
    expire_on_commit=False
)

# Использование
async def get_user(user_id: int):
    async with async_session_factory() as session:  # Берёт соединение из пула
        return await session.get(User, user_id)
    # Соединение возвращается в пул, а не закрывается
```

#### Параметры пула

| Параметр | Описание | Рекомендация |
|----------|----------|--------------|
| `pool_size` | Постоянные соединения | 5-10 для большинства приложений |
| `max_overflow` | Временные соединения сверх pool_size | pool_size × 2 |
| `pool_pre_ping` | Проверка соединения перед использованием | Всегда `True` |
| `pool_recycle` | Время жизни соединения (секунды) | 3600 (1 час) |
| `pool_timeout` | Таймаут ожидания соединения | 30 секунд |

#### Мониторинг пула

```python
from sqlalchemy import event

@event.listens_for(engine.sync_engine, "checkout")
def receive_checkout(dbapi_connection, connection_record, connection_proxy):
    """Логирование при получении соединения из пула."""
    logger.debug(f"Connection checked out: {connection_record}")

@event.listens_for(engine.sync_engine, "checkin")
def receive_checkin(dbapi_connection, connection_record):
    """Логирование при возврате соединения в пул."""
    logger.debug(f"Connection returned to pool: {connection_record}")

# Статистика пула
pool_status = engine.pool.status()
logger.info(f"Pool size: {engine.pool.size()}, Checked out: {engine.pool.checkedout()}")
```

#### Правила

- ✅ **Всегда** настраивать пул с первого дня разработки
- ✅ Использовать `pool_pre_ping=True` для продакшена
- ✅ Мониторить состояние пула в production
- ❌ **Никогда** не создавать соединения вручную в обход пула
- ❌ Не устанавливать слишком большой pool_size (риск перегрузки БД)

### 3. Dependency Lifecycle Management

Универсальный паттерн разделения компонентов по времени жизни. Применим к любому приложению с базой данных (web-серверы, боты, микросервисы, CLI-инструменты).

#### Проблема

```python
# ❌ ПЛОХО: Создаём ВСЁ на каждый запрос
async def handle_request(request):
    http_client = HttpClient()           # Тяжёлый объект — заново
    config = load_config()               # Читаем файл — заново  
    db_session = create_session()        # OK — должен быть свежим
    user_repo = UserRepository(session)  # OK — привязан к сессии
    email_sender = EmailSender()         # Тяжёлый объект — заново
    ...
```

#### Решение: разделение по Scope

```python
# ✅ ХОРОШО: Разделяем по времени жизни

# ══════════════════════════════════════════════════════════════
# APPLICATION SCOPE (Singleton) — создаём ОДИН раз при старте
# ══════════════════════════════════════════════════════════════
class Application:
    def __init__(self):
        # Внешние клиенты и адаптеры
        self.http_client = HttpClient(timeout=30)
        self.email_sender = EmailSender(smtp_config)
        self.telegram_adapter = TelegramAdapter(token)
        
        # Конфигурация
        self.config = load_config()
        self.feature_flags = FeatureFlags()
        
        # Connection pools (НЕ соединения!)
        self.db_engine = create_async_engine(url, pool_size=5)
        self.redis_pool = Redis.from_url(redis_url)
        
        # Stateless сервисы
        self.notification_service = NotificationService(self.email_sender)
        self.metrics = MetricsCollector()

# ══════════════════════════════════════════════════════════════
# REQUEST SCOPE — создаём НА КАЖДЫЙ запрос
# ══════════════════════════════════════════════════════════════
async def handle_request(request, app: Application):
    # Database session (обязательно per-request!)
    async with app.db_engine.session() as session:
        # Repositories (привязаны к сессии)
        user_repo = UserRepository(session)
        order_repo = OrderRepository(session)
        
        # Services с комбинированными зависимостями
        user_service = UserService(
            user_repo=user_repo,                    # Request-scoped
            notification=app.notification_service,  # App-scoped (reuse!)
        )
        
        # Бизнес-логика
        result = await user_service.process(request)
        await session.commit()
        
    return result
```

#### Классификация компонентов

| Scope | Компоненты | Когда создавать |
|-------|------------|-----------------|
| **Application** | HTTP/API клиенты, Email/SMS senders, Message queue clients, Config, Loggers, Connection pools, Stateless services | При старте приложения |
| **Request** | DB Session, Repositories/DAOs, Unit of Work, User context, Transaction, Request-specific cache | На каждый запрос |

#### Паттерн: Dependency Container

```python
from dataclasses import dataclass
from typing import Optional

@dataclass
class AppContainer:
    """Application-scoped dependencies (singleton)."""
    config: Config
    db_engine: AsyncEngine
    http_client: HttpClient
    notification_service: NotificationService
    metrics: MetricsCollector

@dataclass  
class RequestContainer:
    """Request-scoped dependencies."""
    session: AsyncSession
    user_repo: UserRepository
    task_repo: TaskRepository
    current_user: Optional[User] = None

class DependencyManager:
    def __init__(self):
        self._app: Optional[AppContainer] = None
    
    def initialize(self, config: Config) -> None:
        """Call once at application startup."""
        self._app = AppContainer(
            config=config,
            db_engine=create_async_engine(config.db_url),
            http_client=HttpClient(),
            notification_service=NotificationService(),
            metrics=MetricsCollector(),
        )
    
    @asynccontextmanager
    async def request_scope(self):
        """Create request-scoped container."""
        async with self._app.db_engine.session() as session:
            yield RequestContainer(
                session=session,
                user_repo=UserRepository(session),
                task_repo=TaskRepository(session),
            )

# Использование
deps = DependencyManager()
deps.initialize(config)

async def handle_request():
    async with deps.request_scope() as req:
        user = await req.user_repo.get_by_id(1)
        # req.session автоматически закроется
```

#### Правила

- ✅ **Всегда** идентифицируй scope каждого компонента при проектировании
- ✅ Application-scoped компоненты должны быть **thread-safe** / **async-safe**
- ✅ Request-scoped компоненты должны **освобождаться** после запроса
- ✅ Используй **Dependency Injection** для передачи app-scoped в request-scoped
- ❌ **Никогда** не храни request-scoped объекты в application-scoped
- ❌ **Никогда** не создавай тяжёлые stateless объекты на каждый запрос
- ❌ **Никогда** не переиспользуй DB session между запросами

#### Применимость

Этот паттерн универсален и применяется в:

| Тип приложения | Application Scope | Request Scope |
|----------------|-------------------|---------------|
| **Web API** (FastAPI, Django) | App startup | Per HTTP request |
| **Telegram/Discord боты** | Bot startup | Per message/callback |
| **CLI инструменты** | Script startup | Per command execution |
| **Background workers** | Worker startup | Per job/task |
| **Микросервисы** | Service startup | Per RPC call |

### 4. Repository Pattern

```python
from abc import ABC, abstractmethod
from typing import Generic, TypeVar, Optional, List

T = TypeVar('T')

class Repository(ABC, Generic[T]):
    @abstractmethod
    async def create(self, entity: T) -> int:
        """Create entity and return ID."""
        pass
    
    @abstractmethod
    async def get_by_id(self, entity_id: int) -> Optional[T]:
        """Get entity by ID."""
        pass
    
    @abstractmethod
    async def update(self, entity: T) -> None:
        """Update entity."""
        pass
    
    @abstractmethod
    async def delete(self, entity_id: int) -> None:
        """Delete entity by ID."""
        pass
```

### 5. Service Layer

```python
class UserService:
    def __init__(self, user_repository: Repository[User]):
        self.user_repository = user_repository
    
    async def create_user(self, user_data: Dict[str, Any]) -> User:
        """Create user with business logic validation."""
        # Validate business rules
        if await self.user_exists(user_data['username']):
            raise UserAlreadyExistsError(f"User {user_data['username']} already exists")
        
        # Create user
        user = User(
            id=None,
            username=user_data['username'],
            email=user_data['email']
        )
        
        # Save to repository
        user_id = await self.user_repository.create(user)
        user.id = user_id
        
        return user
    
    async def user_exists(self, username: str) -> bool:
        """Check if user exists."""
        user = await self.user_repository.get_by_username(username)
        return user is not None
```

### 6. Error Handling

```python
class ServiceError(Exception):
    """Base exception for service errors."""
    pass

class UserNotFoundError(ServiceError):
    """User not found error."""
    pass

class ValidationError(ServiceError):
    """Validation error."""
    pass

# Usage
async def get_user(user_id: int) -> User:
    try:
        user = await user_repository.get_by_id(user_id)
        if not user:
            raise UserNotFoundError(f"User {user_id} not found")
        return user
    except DatabaseError as e:
        raise ServiceError(f"Database error: {e}") from e
```

### 7. Configuration Management

```python
from pydantic import BaseSettings
from typing import Optional

class Settings(BaseSettings):
    database_url: str
    secret_key: str
    debug: bool = False
    log_level: str = "INFO"
    
    class Config:
        env_file = ".env"
        env_file_encoding = "utf-8"

# Usage
settings = Settings()
```

## 🚀 Getting Started Checklist

### Day 1: Environment Setup

- [ ] Install Python 3.11+
- [ ] Install Poetry
- [ ] Set up IDE with extensions
- [ ] Create project structure
- [ ] Configure tools (Black, MyPy, Pytest)

### Day 2: First Feature

- [ ] Write feature description
- [ ] Create test for user scenario
- [ ] Implement core functionality
- [ ] Add integration tests
- [ ] Document changes

### Day 3: Advanced Patterns

- [ ] Implement repository pattern
- [ ] Add service layer
- [ ] Create comprehensive error handling
- [ ] Write end-to-end tests
- [ ] Set up CI/CD pipeline

This development guide provides a solid foundation for building modern Python applications with async-first architecture, comprehensive testing, and clean code principles.
