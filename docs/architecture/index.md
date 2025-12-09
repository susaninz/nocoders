# 🏗️ Архитектура

> Архитектурные паттерны и принципы проектирования

---

## О чём этот раздел

Как строить приложения, которые легко поддерживать и развивать. Clean Architecture, паттерны, SOLID — без академического занудства, с примерами из реальных проектов.

**Для кого:**

- Вайбкодеры, которые хотят писать чистый код
- Те, чьи проекты выросли из скриптов
- Начинающие архитекторы

---

## Содержание

| Раздел | Описание |
|--------|----------|
| [Clean Architecture](clean-architecture.md) | Чистая архитектура и разделение слоёв |
| [Dependency Lifecycle](dependency-lifecycle.md) | Управление жизненным циклом зависимостей |
| [State Machine](state-machine.md) | Конечные автоматы для управления состоянием |
| [Repository Pattern](repository-pattern.md) | Абстракция доступа к данным |

---

## 🎯 Ключевые принципы

### SOLID

| Принцип | Описание |
|---------|----------|
| **S**ingle Responsibility | Один класс — одна ответственность |
| **O**pen/Closed | Открыт для расширения, закрыт для изменения |
| **L**iskov Substitution | Подтипы должны быть заменяемы |
| **I**nterface Segregation | Много специфичных интерфейсов лучше одного общего |
| **D**ependency Inversion | Зависимость от абстракций, не от реализаций |

### Другие принципы

- **KISS** — Keep It Simple, Stupid
- **YAGNI** — You Aren't Gonna Need It
- **DRY** — Don't Repeat Yourself
- **Composition over Inheritance** — Композиция вместо наследования

---

## 🏛️ Слои Clean Architecture

```
┌─────────────────────────────────────────┐
│           Presentation Layer            │
│     (UI, Controllers, API Handlers)     │
├─────────────────────────────────────────┤
│           Application Layer             │
│    (Use Cases, Services, Workflows)     │
├─────────────────────────────────────────┤
│             Domain Layer                │
│   (Entities, Business Rules, Events)    │
├─────────────────────────────────────────┤
│          Infrastructure Layer           │
│  (Database, External APIs, Frameworks)  │
└─────────────────────────────────────────┘

Направление зависимостей: снаружи → внутрь
```

---

## 📐 Когда применять

| Сложность проекта | Рекомендация |
|-------------------|--------------|
| Простой скрипт | Не нужно, overcomplicated |
| Небольшой проект | Repository + Service layer |
| Средний проект | Clean Architecture lite |
| Сложный проект | Full Clean Architecture |
| Enterprise | Clean Architecture + DDD |

---

*Обновлено: 2025-12-09*
