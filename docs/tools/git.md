# Git

Команды, workflow и best practices.

---

## 🔧 Базовые команды

### Ежедневные операции

```bash
# Статус
git status
git diff

# Коммит
git add .
git commit -m "feat: add user authentication"

# Pull/Push
git pull origin main
git push origin feature/my-feature

# Ветки
git checkout -b feature/new-feature
git checkout main
git branch -d feature/old-feature
```

### История

```bash
# Лог
git log --oneline -10
git log --graph --oneline --all

# Поиск коммита
git log --grep="fix bug"
git log -S "function_name"  # Поиск по содержимому
```

---

## 🌿 Git Flow

### Основные ветки

```
main (production)
  │
  ├── develop (integration)
  │     │
  │     ├── feature/user-auth
  │     ├── feature/task-management
  │     └── bugfix/login-error
  │
  └── hotfix/critical-bug
```

### Workflow

```bash
# 1. Создать feature ветку от develop
git checkout develop
git pull
git checkout -b feature/my-feature

# 2. Работать над фичей
git add .
git commit -m "feat: implement feature"

# 3. Обновить из develop
git fetch origin
git rebase origin/develop

# 4. Push и создать PR
git push origin feature/my-feature
# Создать Pull Request в GitHub/GitLab

# 5. После merge — удалить ветку
git checkout develop
git pull
git branch -d feature/my-feature
```

---

## 📝 Conventional Commits

### Формат

```
<type>(<scope>): <description>

[optional body]

[optional footer]
```

### Типы

| Тип | Описание |
|-----|----------|
| `feat` | Новая функциональность |
| `fix` | Исправление бага |
| `docs` | Документация |
| `style` | Форматирование (без изменения логики) |
| `refactor` | Рефакторинг |
| `test` | Тесты |
| `chore` | Обслуживание (CI, зависимости) |

### Примеры

```bash
git commit -m "feat(auth): add JWT authentication"
git commit -m "fix(api): handle null response from external service"
git commit -m "docs: update README with installation steps"
git commit -m "refactor(user): extract validation to separate module"
git commit -m "test(task): add integration tests for task creation"
```

---

## 🔄 Полезные операции

### Stash

```bash
# Сохранить изменения
git stash
git stash save "WIP: user feature"

# Восстановить
git stash pop
git stash apply stash@{0}

# Список
git stash list
```

### Rebase

```bash
# Интерактивный rebase (последние 3 коммита)
git rebase -i HEAD~3

# Squash коммитов в PR
git rebase -i origin/main
# Заменить 'pick' на 'squash' для объединения
```

### Отмена изменений

```bash
# Отменить незакоммиченные изменения в файле
git checkout -- file.py

# Отменить staged изменения
git reset HEAD file.py

# Отменить последний коммит (сохранить изменения)
git reset --soft HEAD~1

# Отменить последний коммит (удалить изменения)
git reset --hard HEAD~1
```

---

## ⚙️ Конфигурация

### .gitignore

```gitignore
# Python
__pycache__/
*.py[cod]
*.egg-info/
dist/
build/
.eggs/
venv/
.venv/

# IDE
.idea/
.vscode/
*.swp

# Environment
.env
.env.local

# Database
*.db
*.sqlite3

# Logs
*.log

# OS
.DS_Store
Thumbs.db
```

### Алиасы

```bash
# ~/.gitconfig
[alias]
    st = status
    co = checkout
    br = branch
    ci = commit
    lg = log --graph --oneline --all
    unstage = reset HEAD --
    last = log -1 HEAD
```

---

## 🛡️ Best Practices

- ✅ Маленькие, атомарные коммиты
- ✅ Понятные сообщения коммитов
- ✅ Не коммитить секреты и .env файлы
- ✅ Регулярно pull из основной ветки
- ✅ Code review через Pull Requests
- ❌ Не делать force push в общие ветки
- ❌ Не коммитить сгенерированные файлы

