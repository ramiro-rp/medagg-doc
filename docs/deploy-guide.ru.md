# Руководство по развёртыванию

В этом документе описывается, как развернуть **Medical Imaging Dataset Aggregator**
в простом односерверном окружении (MVP).

---

## 1. Архитектура (кратко)

Компоненты:

- **Backend** — Django + Django REST Framework
- **Database** — PostgreSQL
- **Frontend** — Vite + React + Ant Design (SPA)
- **Файловое хранилище** — локальная файловая система (опционально)

---

## 2. Требования

- Linux / macOS / WSL2 (для разработки)
- Docker и docker compose
- Git (если клонируете репозитории)

---

## 3. Развёртывание backend-а (Docker)

### 3.1 Клонирование

```bash
git clone <backend-repo-url>
cd medagg-backend
```

### 3.2 Файлы окружения

`docker-compose.yml` ожидает два файла:

- `./.env/database.env`
- `./.env/settings.env`

Если в репозитории нет шаблонов, создайте папку и начните с минимального примера.

**`.env/database.env`** (пример):

```bash
DB_ENGINE=postgresql
DB_HOST=db
DB_PORT=5432
DB_NAME=medagg
DB_USER=medagg
DB_PASSWORD=change-me
```

**`.env/settings.env`** (пример):

```bash
DJANGO_SECRET_KEY=change-me
DJANGO_DEBUG_MODE=0
DJANGO_ALLOWED_HOSTS=127.0.0.1,localhost
```

### 3.3 Запуск

Если в проекте есть вспомогательный скрипт:

```bash
python configure.py --docker
```

Или напрямую через compose:

```bash
docker compose up -d
```

После запуска:

- API: `http://<server-host>:8000/api/v1/`
- Админка: `http://<server-host>:8000/admin/`
- Browsable API логин (dev): `http://<server-host>:8000/api-auth/`

---

## 4. Frontend

### 4.1 Dev режим

Frontend читает `VITE_API_URL` — базовый URL backend API, например:

```bash
VITE_API_URL=http://127.0.0.1:8000/api/v1
```

> Если frontend запускается на другом origin (Vite dev server), может понадобиться CORS или dev proxy.
> Конкретная настройка зависит от выбранного способа развёртывания в команде.

### 4.2 Production (static build)

Для простого демо frontend можно собрать и отдавать через Nginx (см. README фронтенд‑репозитория).
