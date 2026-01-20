# Руководство по развёртыванию

В этом документе описано, как развернуть
**Medical Imaging Dataset Aggregator** в простом одноузловом окружении.

---

## 1. Обзор архитектуры

Компоненты системы:

- **Backend** – Django + Django REST Framework;
- **Database** – PostgreSQL;
- **Frontend** – Vite + React + Ant Design (SPA);
- **Хранилище файлов** – локальная файловая система для ZIP-архивов.

В MVP все компоненты могут быть развернуты на **одной машине**.
Официальный путь развёртывания для воспроизводимости — Docker / docker-compose.

> Примечание по модулю поиска: backend ожидает пакет `medagg-search` в `src/libs/medsearch/`.
> Если вы клонируете backend репозиторий, инициализируйте подмодуль через git submodules
> (или скопируйте пакет в этот путь) перед запуском API.

---

## 2. Требования

- Linux сервер/рабочая станция (WSL2 / macOS подходят для разработки);
- Docker и docker-compose (или Docker + compose plugin);
- Git.

> Примечание (локальные интеграционные тесты без Docker): официальный MVP-путь развёртывания
> Docker-based для воспроизводимости.
> Если Docker недоступен в вашей среде (например, Windows без WSL2/Docker),
> backend и frontend можно запустить локально, следуя README каждого репозитория.
> Этот документ описывает стандартизированный одноузловой Docker-сетап.

---

## 3. Развёртывание backend (кратко)

Типовой сценарий:

```bash
# Референсный backend репозиторий (при необходимости используйте форк команды)
git clone https://github.com/b-barsky/medagg-backend.git
cd medagg-backend

# Скопируйте пример env-файла, если он присутствует
cp .env.example .env

# Настройте .env (подключение к БД, secret key и т.п.)

# Запустите сервисы
docker compose up -d  # или: docker-compose up -d
```

После старта:

- backend API доступен по адресу `http://<server-host>:8000/api/v1/`;
- browsable API / схема могут быть доступны на `/api/`, `/api/schema/`
  (в зависимости от конфигурации).

Миграции (если не применяются автоматически):

```bash
docker compose exec backend python manage.py migrate
```

---

## 4. Развёртывание frontend (кратко)

### Режим разработки

```bash
# Frontend репозиторий
git clone https://github.com/Nikitmen/frontend-DataHive.git
cd frontend-DataHive

npm install
npm run dev
```

По умолчанию Vite dev server доступен на `http://localhost:5173/` и обращается к backend API.

### Простой production-вариант

1. Соберите фронтенд:

   ```bash
   npm run build
   ```

2. Раздавайте каталог `dist/` через `nginx`, `caddy` или другой HTTP-сервер.

Важно:

- задать корректный base URL backend API для фронтенда;
- настроить CORS на backend, чтобы разрешить запросы с origin фронтенда.

---

## 5. Конфигурация

Ключевые параметры (обычно через `.env`):

- **Backend**
  - host/port/name/user/password для БД;
  - Django secret key;
  - allowed hosts;
  - CORS;
  - путь к директории ZIP-архивов (собранные коллекции).
- **PostgreSQL**
  - каталог/volume для данных БД.
- **Frontend**
  - base URL API (фронтенд ожидает `VITE_API_URL`).

Точные имена переменных смотрите в README каждого репозитория.

---

## 6. Резервное копирование и восстановление

### 6.1 PostgreSQL

Пример backup:

```bash
pg_dump -U <db_user> <db_name> > backup.sql
```

Пример restore:

```bash
psql -U <db_user> <db_name> < backup.sql
```

В Docker окружениях эти команды выполняются через `docker compose exec db ...`,
как описано в backend README.

### 6.2 Сформированные архивы

Собранные коллекции сохраняются как ZIP-файлы на файловой системе хоста
или в примонтированном Docker volume. Для бэкапа можно:

- периодически архивировать/копировать эту директорию;
- использовать внешнюю систему резервного копирования.

Если включена политика ретенции (удаление архивов старше N дней),
заранее определите, требуется ли долгосрочное хранение.

---

## 7. Мониторинг и логи

Базовый мониторинг через логи:

```bash
docker compose logs -f backend
docker compose logs -f db
```

Для более серьёзных сетапов можно:

- настроить HTTPS через reverse proxy (nginx, caddy);
- добавить метрики и дашборды (Prometheus, Grafana и т.п.).

---

## 8. Ограничения MVP-развёртывания

- одноузловая среда (без кластеризации и горизонтального масштабирования);
- локальное файловое хранилище для ZIP-архивов;
- минимальные мониторинг и алёртинг «из коробки».
