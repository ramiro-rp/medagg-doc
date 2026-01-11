# Модули системы

Этот документ даёт обзор основных модулей системы на высоком уровне.
Для настройки и технических деталей см. `README.md` соответствующих репозиториев.

---

## Репозитории

- **Backend API** (`medagg-backend`)
  - GitHub: <https://github.com/b-barsky/medagg-backend>

- **Модуль поиска** (`medagg-search`)
  - GitHub: <https://github.com/ESBehtev/medagg-search>

- **Frontend** (`frontend-DataHive`)
  - GitHub: <https://github.com/Nikitmen/frontend-DataHive>

---

## Backend (`medagg-backend`)

- Django + DRF API (базовый путь: `/api/v1/`)
- Каталог датасетов (список, фильтры, детали)
- Эндпоинт поиска, который делегирует в модуль поиска

### Генерация README для датасета (feature-ветка)

В backend есть feature (обсуждалась в командном чате), которая генерирует текст README для датасета на основе метаданных датасета.

- Ветка backend, указанная командой: `feature/search`
- Быстрая проверка (как описано разработчиком):
  - Убедиться, что папка `data/` пустая
  - Запустить: `python configure.py --docker`
  - Открыть Django shell: `docker-compose exec medagg-app python manage.py shell`
  - Создать тестовый датасет и проверить, что генерация README отработала

(Если для хранения сгенерированного README потребуется изменение схемы БД, это фиксируется в чек-листе ревью.)

---

## Модуль поиска (`medagg-search`)

- Парсинг / нормализация запроса
- Сопоставление с таксономией
- Ранжирование / скоринг (если включено интеграцией backend)

---

## Frontend (`frontend-DataHive`)

- UI для поиска датасетов и просмотра результатов/деталей
- Обращается к backend API, адрес задаётся через `VITE_API_URL`
