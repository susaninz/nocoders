# Тестирование в Python

TDD, pytest, моки и фикстуры.

---

## 🎯 Test-Driven Development (TDD)

### Цикл Red-Green-Refactor

```
1. 🔴 RED    — Напиши падающий тест
2. 🟢 GREEN — Напиши минимальный код для прохождения
3. 🔵 REFACTOR — Улучши код, тесты должны проходить
```

### Пример TDD

```python
# 1. Сначала тест (падает)
def test_calculate_discount():
    assert calculate_discount(100, 10) == 90

# 2. Минимальная реализация
def calculate_discount(price: float, percent: float) -> float:
    return price - (price * percent / 100)

# 3. Рефакторинг (если нужен)
def calculate_discount(price: float, percent: float) -> float:
    """Calculate price after discount."""
    discount = price * percent / 100
    return round(price - discount, 2)
```

---

## 🧪 Pytest

### Структура тестов

```
tests/
├── conftest.py          # Общие фикстуры
├── unit/
│   ├── test_services.py
│   └── test_models.py
├── integration/
│   └── test_database.py
└── e2e/
    └── test_workflows.py
```

### Базовый тест

```python
import pytest

def test_user_creation():
    """Test creating a new user."""
    user = User(username="john", email="john@example.com")
    
    assert user.username == "john"
    assert user.email == "john@example.com"
    assert user.is_active is True  # Default value
```

### Async тесты

```python
import pytest

@pytest.mark.asyncio
async def test_fetch_user():
    """Test async user fetching."""
    user = await user_service.get_by_id(1)
    
    assert user is not None
    assert user.id == 1
```

---

## 📦 Фикстуры

### Базовые фикстуры

```python
# conftest.py
import pytest

@pytest.fixture
def sample_user():
    """Create a sample user for testing."""
    return User(
        id=1,
        username="testuser",
        email="test@example.com"
    )

@pytest.fixture
async def db_session():
    """Create async database session."""
    async with async_session_factory() as session:
        yield session
        await session.rollback()
```

### Использование фикстур

```python
def test_user_display_name(sample_user):
    """Test using fixture."""
    assert sample_user.display_name == "Testuser"

@pytest.mark.asyncio
async def test_save_user(db_session, sample_user):
    """Test with multiple fixtures."""
    db_session.add(sample_user)
    await db_session.commit()
    
    saved = await db_session.get(User, sample_user.id)
    assert saved is not None
```

---

## 🎭 Моки

### Мокирование зависимостей

```python
from unittest.mock import AsyncMock, MagicMock, patch

@pytest.mark.asyncio
async def test_send_notification():
    """Test with mocked external service."""
    # Создаём мок
    mock_sender = AsyncMock()
    mock_sender.send.return_value = True
    
    service = NotificationService(sender=mock_sender)
    result = await service.notify_user(user_id=1, message="Hello")
    
    assert result is True
    mock_sender.send.assert_called_once()

@pytest.mark.asyncio
async def test_with_patch():
    """Test with patch decorator."""
    with patch('app.services.email.send_email', new_callable=AsyncMock) as mock:
        mock.return_value = True
        
        result = await user_service.send_welcome_email(user_id=1)
        
        assert result is True
        mock.assert_called_once()
```

---

## 📊 Покрытие кода

```bash
# Запуск с покрытием
pytest --cov=app --cov-report=html

# Минимальное покрытие
pytest --cov=app --cov-fail-under=80
```

### pytest.ini

```ini
[pytest]
asyncio_mode = auto
testpaths = tests
python_files = test_*.py
python_classes = Test*
python_functions = test_*
addopts = -v --tb=short --cov=app --cov-report=term-missing
```

---

## ✅ Чеклист хорошего теста

- [ ] Тест имеет понятное название
- [ ] Один тест проверяет одну вещь
- [ ] Тест изолирован (не зависит от других)
- [ ] Тест детерминирован (всегда одинаковый результат)
- [ ] Тест быстрый
- [ ] Используются фикстуры для setup
- [ ] Моки для внешних зависимостей

