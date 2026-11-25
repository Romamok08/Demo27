# Итоговый отчёт по ревью проекта Demo27

**Дата:** 27.11.2026
**Ревьюер:** Данильченков Роман
**Репозиторий:** https://github.com/Malenikajkat/Demo27 (fork: Romamok08/Demo27)
**Ветка ревью:** review/danilchenkov
**Период:** 5 учебных дней, 36 часов
**Модуль:** «Ревьюирование»

---

## 1. Общая информация

Проект Demo27 — Flask-приложение для учёта заказов (клиенты, продукты, материалы,
операции, спецификации, заказы на продажу и производство, заметки, админка).
Архитектура слоистая: blueprints/views → services → repositories → models,
с DTO для передачи данных. БД — PostgreSQL, фронтенд — статические HTML в `web/`.

**Объём ревью:** 5 отчётов, 1 сводная таблица, 5 правок, pull request.

---

## 2. Артефакты ревью

| День | Артефакт | Файл |
|---|---|---|
| 1 | Карта репозитория | `docs/review/repo_map.md` |
| 1 | Персональный чек-лист | `docs/review/checklist.md` |
| 2 | Отчёт по стилю и читаемости | `docs/review/style_report.md` + `style_report.txt` |
| 3 | Отчёт по логике, БД и тестам | `docs/review/logic_report.md` |
| 4 | Отчёт по безопасности | `docs/review/security_report.md` |
| 4 | Отчёт по зависимостям | `docs/review/deps_report.md` |
| 5 | Итоговый отчёт | `docs/review/review_report.md` |

---

## 3. Сводная таблица замечаний

### 3.1. Критичные (требуют немедленного исправления)

| № | Файл / строка | Категория | Замечание | Предложение | Статус |
|---|---|---|---|---|---|
| 1 | `app/blueprints/auth.py:register` | Безопасность | `role` берётся из запроса → любой может стать администратором | Убрать `role` из публичной регистрации | ✅ Исправлено (`9ddf3d7`) |
| 2 | `app/config.py:59` | Безопасность | `SECRET_KEY` с дефолтом `production-is-system-secret-key-change-me` | Убрать дефолт, падать при отсутствии env | ✅ Исправлено (`9ecbeb1`) |
| 3 | `app/config.py:53` | Безопасность | Пароль БД `12345` захардкожен | Убрать дефолт, только из env | ✅ Исправлено (`9ecbeb1`) |
| 4 | `run.py` | Безопасность | `host='0.0.0.0', debug=True` | `host='127.0.0.1'`, debug из env | ✅ Исправлено (`e2afa72`) |
| 5 | `web/login.html` + все POST | Безопасность | Нет CSRF-защиты | Подключить `Flask-WTF` + `CSRFProtect` | ⏳ В бэклоге (частично: `Flask-WTF` добавлен в `requirements.txt`) |
| 6 | `requirements.txt` | Зависимости | `psycopg2-binary` вместо `psycopg` (v3) → проект не стартует | Заменить на `psycopg[binary]` | ✅ Исправлено (`645ed0a`) |
| 7 | `app/__init__.py` | Логика | `db.create_all()` в продакшене | Только в DEBUG, в проде — миграции | ✅ Исправлено (`7facf31`) |
| 8 | `requirements.txt` | Тесты | Нет `pytest` — тесты не запускаются | Добавить `pytest` | ✅ Исправлено (`645ed0a`) |

### 3.2. Важные

| № | Файл | Категория | Замечание | Предложение |
|---|---|---|---|---|
| 1 | `app/views/client_view.py:post` | Логика | Нет валидации `client_type`, длины, ИНН, телефона | Добавить проверки, возвращать 400 |
| 2 | `app/views/client_view.py:put` | Логика | `client_type` не валидируется → 500 вместо 400 | Валидировать до вызова сервиса |
| 3 | `app/blueprints/auth.py:login` | Логика | Нет проверки типов `login`/`password` | `isinstance(...)` |
| 4 | `app/blueprints/auth.py` | Логика | Нет rollback при ошибке `commit()` | `try/except` с `db.session.rollback()` |
| 5 | `app/views/production_order_view.py` | Безопасность | `sqlalchemy.text` — проверить параметризацию | Все параметры — через `:param` |
| 6 | `database/schema.sql` vs `app/__init__.py` | БД | Два источника истины: `schema.sql` и `db.create_all()` | Использовать миграции (Flask-Migrate) |
| 7 | `database/schema.sql` | БД | Нет `ON DELETE` для `notes.id_user`, `sales_orders.client_id` | Добавить `ON DELETE CASCADE/RESTRICT` |
| 8 | `database/order_cost_query.sql` | БД | Функция `calculate_order_cost` создаётся в БД, нет миграции | Добавить в миграции |
| 9 | `tests/` | Тесты | Покрыты только `auth` и `notes` (2 из 10+ доменов) | Добавить тесты на остальные домены |
| 10 | `tests/` | Тесты | SQLite для тестов vs PostgreSQL в проде | Использовать PostgreSQL в тестах |
| 11 | `web/login.html` | Фронтенд | Инлайновые `<style>` и `<script>` (~400 строк) | Вынести в `static/css/`, `static/js/` |
| 12 | `docs/` | Документация | Нет `README.md`, нет инструкции по запуску | Добавить README |

### 3.3. Рекомендации

| № | Файл | Категория | Замечание |
|---|---|---|---|
| 1 | `app/views/sales_order_view.py:431` | Стиль | Строка длиной 197 символов |
| 2 | `app/__init__.py` | Логика | 20 локальных переменных в `create_app()` — разбить на хелперы |
| 3 | `app/puzzle_captcha.py` | Стиль | `else`/`elif` после `return` |
| 4 | `app/views/production_order_view.py:73` | Стиль | Повторный импорт `db` |
| 5 | `database/schema.sql` | БД | `SERIAL` для `users` vs `UUID` для остальных |
| 6 | `database/schema.sql` | БД | Избыточный индекс `idx_product_prices_product` |
| 7 | `tests/conftest.py` | Тесты | Дублирование фикстур `app`/`client` |
| 8 | `app/blueprints/auth.py:register` | Стиль | `data = request.get_json()` дважды |
| 9 | `docs/` | Документация | Нет `er_diagram.pdf` |
| 10 | `image/` в корне | Структура | Скриншоты в корне — перенести в `docs/images/` |
| 11 | `scripts/` | Структура | Каталог отсутствует |
| 12 | `app/views/*`, `app/blueprints/*` | Документация | Нет докстрингов у многих публичных методов |

---

## 4. Статистика

| Метрика | Значение |
|---|---|
| Всего замечаний | 8 критичных + 12 важных + 12 рекомендаций = **32** |
| Исправлено правками | **5** (критичные 1, 2, 3, 4, 6, 7, 8 — часть в одном коммите) |
| Коммитов `review(...)` | 4 (дни 1–4) |
| Коммитов `fix:` | 5 |
| Отчётов | 6 (`repo_map`, `checklist`, `style`, `logic`, `security`, `deps`, `review`) |
| Скор pylint | 8.84/10 |
| Нарушений flake8 | 142 |

---

## 5. Внесённые правки (коммиты `fix:`)

| № | Коммит | Файл | Что исправлено |
|---|---|---|---|
| 1 | `645ed0a` | `requirements.txt` | `psycopg[binary]` вместо `psycopg2-binary`, добавлены `pytest` и `Flask-WTF`, версии зафиксированы `==` |
| 2 | `9ecbeb1` | `app/config.py` | Убраны дефолтные `SECRET_KEY` и `DATABASE_PASSWORD`, добавлена проверка env |
| 3 | `9ddf3d7` | `app/blueprints/auth.py` | Запрещено повышение привилегий через `role` при регистрации |
| 4 | `e2afa72` | `run.py` | `host='127.0.0.1'`, debug из env |
| 5 | `7facf31` | `app/__init__.py` | `db.create_all()` только в DEBUG |

---

## 6. Выводы и рекомендации

### Сильные стороны проекта
- Слоистая архитектура (blueprints → services → repositories → models) с чётким разделением ответственности.
- ORM-запросы SQLAlchemy без конкатенации SQL — риск SQL-инъекций минимален.
- Пароли хешируются (`generate_password_hash`/`check_password_hash`).
- Jinja2 экранирует переменные — XSS через шаблоны не выявлен.
- Хорошее покрытие тестами `auth` и `notes` (позитивные и негативные сценарии).
- `CHECK`-ограничения и индексы в `schema.sql`.
- `.env` в `.gitignore`, секреты не в репозитории.

### Что исправлено в ходе ревью
- Повышение привилегий через `role` при регистрации (критично).
- Дефолтные `SECRET_KEY` и пароль БД (критично).
- `debug=True` на `0.0.0.0` (критично).
- Неверный драйвер БД (`psycopg2-binary` → `psycopg`).
- `db.create_all()` в продакшене.
- Отсутствие `pytest` и `Flask-WTF` в зависимостях.

### Что осталось в бэклоге
1. **CSRF-защита:** подключить `Flask-WTF` + `CSRFProtect`, добавить `{{ csrf_token() }}` в формы (критично для продакшена).
2. **Валидация входных данных** в маршрутах (`client_type`, длина, ИНН, телефон).
3. **Тесты:** покрыть CRUD остальных доменов, добавить тест на `register` с `role="Администратор"` (должен игнорироваться).
4. **Миграции:** перейти на Flask-Migrate, убрать `db.create_all()` полностью.
5. **Рефакторинг `create_app()`:** разбить на хелперы.
6. **Фронтенд:** вынести инлайновые `<style>`/`<script>` из `login.html` в `static/`.
7. **Документация:** добавить `README.md`, `er_diagram.pdf`, `scripts/`.
8. **Автоформатирование:** `black` + `isort` + `pre-commit` (устранит 90% flake8 W293/E501).

### Общая оценка
Проект имеет **хорошую архитектурную основу** и **читаемый код** (pylint 8.84/10), но содержит **критичные уязвимости безопасности**, которые частично исправлены в ходе ревью. Основные направления дальнейшей работы — CSRF-защита, валидация входных данных, расширение тестового покрытия и переход на миграции.

---

## 7. Ссылки

- Pull Request: `<ссылка будет добавлена после создания PR>`
- Ветка: `review/danilchenkov`
- Fork: https://github.com/Romamok08/Demo27