# Async паттерны в Python

Асинхронное программирование с asyncio.

---

## 🎯 Основы async/await

### Когда использовать async

| Операция | Async нужен? | Почему |
|----------|--------------|--------|
| HTTP запросы | ✅ Да | I/O bound |
| Работа с БД | ✅ Да | I/O bound |
| Чтение файлов | ✅ Да | I/O bound |
| CPU-вычисления | ❌ Нет | CPU bound (используй multiprocessing) |
| Память/переменные | ❌ Нет | Мгновенно |

---

## 📝 Базовые паттерны

### Простой async/await

```python
import asyncio

async def fetch_data(url: str) -> dict:
    """Асинхронный запрос данных."""
    async with aiohttp.ClientSession() as session:
        async with session.get(url) as response:
            return await response.json()

# Запуск
result = asyncio.run(fetch_data("https://api.example.com"))
```

### Параллельное выполнение с gather

```python
async def fetch_all(urls: list[str]) -> list[dict]:
    """Параллельные запросы."""
    tasks = [fetch_data(url) for url in urls]
    results = await asyncio.gather(*tasks, return_exceptions=True)
    
    # Фильтруем ошибки
    return [r for r in results if not isinstance(r, Exception)]
```

### Таймауты

```python
async def fetch_with_timeout(url: str, timeout: float = 5.0) -> dict:
    """Запрос с таймаутом."""
    try:
        return await asyncio.wait_for(
            fetch_data(url),
            timeout=timeout
        )
    except asyncio.TimeoutError:
        raise TimeoutError(f"Request to {url} timed out")
```

---

## 🔄 Context Managers

### Async context manager

```python
from contextlib import asynccontextmanager

@asynccontextmanager
async def database_session():
    """Async context manager для сессии БД."""
    session = await create_session()
    try:
        yield session
        await session.commit()
    except Exception:
        await session.rollback()
        raise
    finally:
        await session.close()

# Использование
async with database_session() as session:
    user = await session.get(User, 1)
```

---

## ⚠️ Частые ошибки

### ❌ Блокирующий код в async

```python
# ПЛОХО: time.sleep блокирует event loop
async def bad_example():
    time.sleep(5)  # ❌ Блокирует всё!

# ХОРОШО: asyncio.sleep не блокирует
async def good_example():
    await asyncio.sleep(5)  # ✅ Другие задачи могут работать
```

### ❌ Забыть await

```python
# ПЛОХО: Получаем coroutine, не результат
result = fetch_data(url)  # ❌ <coroutine object>

# ХОРОШО
result = await fetch_data(url)  # ✅ Фактический результат
```

---

## 📚 Дополнительно

- [asyncio документация](https://docs.python.org/3/library/asyncio.html)
- [aiohttp](https://docs.aiohttp.org/)
- [SQLAlchemy Async](https://docs.sqlalchemy.org/en/20/orm/extensions/asyncio.html)

