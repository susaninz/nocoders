# Dependency Lifecycle Management

Универсальный паттерн управления жизненным циклом зависимостей.

---

## 🎯 Основная идея

Разделяй компоненты по времени жизни:

- **Application Scope** — создаются один раз при старте
- **Request Scope** — создаются на каждый запрос

---

## ❌ Проблема

```python
# ПЛОХО: Создаём ВСЁ на каждый запрос
async def handle_request(request):
    http_client = HttpClient()           # 50ms на создание
    config = load_config()               # Чтение файла
    db_session = create_session()        # OK
    email_sender = EmailSender()         # Инициализация SMTP
    ...
```

**Результат**: Медленно, ресурсоёмко, много мусора для GC.

---

## ✅ Решение

```python
# ХОРОШО: Разделяем по scope

class Application:
    """Application Scope — создаётся один раз."""
    
    def __init__(self):
        # Тяжёлые компоненты
        self.http_client = HttpClient(timeout=30)
        self.email_sender = EmailSender(smtp_config)
        self.config = load_config()
        
        # Connection pools (не соединения!)
        self.db_engine = create_async_engine(url, pool_size=5)
        
        # Stateless сервисы
        self.notification_service = NotificationService(self.email_sender)


async def handle_request(request, app: Application):
    """Request Scope — создаётся на каждый запрос."""
    
    async with app.db_engine.session() as session:
        # Только то, что привязано к сессии
        user_repo = UserRepository(session)
        
        # Комбинируем app-scoped и request-scoped
        service = UserService(
            repo=user_repo,                    # Request-scoped
            notifier=app.notification_service  # App-scoped (reuse!)
        )
        
        return await service.process(request)
```

---

## 📊 Классификация компонентов

### Application Scope (Singleton)

| Компонент | Почему singleton |
|-----------|------------------|
| HTTP/API клиенты | Тяжёлая инициализация, connection pool |
| Email/SMS senders | Установка соединения с сервером |
| Config | Чтение файлов, парсинг |
| Loggers | Настройка handlers |
| Connection pools | Управление соединениями |
| Feature flags | Кэширование значений |
| Metrics collectors | Агрегация метрик |

### Request Scope (Per-request)

| Компонент | Почему per-request |
|-----------|-------------------|
| DB Session | Изоляция транзакций |
| Repositories | Привязаны к сессии |
| Unit of Work | Scope транзакции |
| User context | Данные текущего пользователя |
| Request cache | Кэш в рамках запроса |

---

## 🏗️ Паттерн: Dependency Container

```python
from dataclasses import dataclass
from contextlib import asynccontextmanager

@dataclass
class AppContainer:
    """Application-scoped dependencies."""
    config: Config
    db_engine: AsyncEngine
    http_client: HttpClient
    notification_service: NotificationService

@dataclass
class RequestContainer:
    """Request-scoped dependencies."""
    session: AsyncSession
    user_repo: UserRepository
    task_repo: TaskRepository
    current_user: User | None = None


class DependencyManager:
    def __init__(self):
        self._app: AppContainer | None = None
    
    def initialize(self, config: Config) -> None:
        """Вызвать один раз при старте."""
        self._app = AppContainer(
            config=config,
            db_engine=create_async_engine(config.db_url),
            http_client=HttpClient(),
            notification_service=NotificationService(),
        )
    
    @asynccontextmanager
    async def request_scope(self):
        """Создать request-scoped контейнер."""
        async with self._app.db_engine.session() as session:
            yield RequestContainer(
                session=session,
                user_repo=UserRepository(session),
                task_repo=TaskRepository(session),
            )


# Использование
deps = DependencyManager()
deps.initialize(config)

async def handle_message(message):
    async with deps.request_scope() as req:
        user = await req.user_repo.get_by_id(message.user_id)
        # session автоматически закроется
```

---

## 📐 Применимость

| Тип приложения | App Scope | Request Scope |
|----------------|-----------|---------------|
| **Web API** | App startup | Per HTTP request |
| **Telegram бот** | Bot startup | Per message/callback |
| **CLI утилита** | Script start | Per command |
| **Background worker** | Worker start | Per job |
| **Микросервис** | Service start | Per RPC call |

---

## ✅ Правила

- ✅ Всегда идентифицируй scope каждого компонента
- ✅ App-scoped должны быть thread-safe / async-safe
- ✅ Request-scoped должны освобождаться после запроса
- ✅ Используй Dependency Injection
- ❌ Не храни request-scoped в app-scoped
- ❌ Не создавай тяжёлые stateless объекты на каждый запрос
- ❌ Не переиспользуй DB session между запросами

---

## 📈 Выигрыш

| Метрика | Без паттерна | С паттерном |
|---------|--------------|-------------|
| Время отклика | 100ms+ | 10-20ms |
| Память | Растёт | Стабильная |
| GC давление | Высокое | Низкое |
| Масштабируемость | Плохая | Хорошая |

