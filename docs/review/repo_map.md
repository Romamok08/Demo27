# Карта репозитория Demo27

**Дата:** 23.11.2026
**Ревьюер:** Данильченков Роман
**Репозиторий:** https://github.com/Malenikajkat/Demo27
**Ветка ревью:** review/danilchenkov

## 1. Назначение проекта

Flask-приложение для учёта заказов/CRM (клиенты, продукты, материалы, операции,
спецификации, заказы на продажу и производство, заметки, админка).
Архитектура — слоистая: blueprints/views (маршруты) → services (бизнес-логика)
→ repositories (доступ к БД) → models (SQLAlchemy). DTO — для передачи данных
между слоями. Фронтенд — статические HTML + styles.css в web/.
БД — PostgreSQL.

## 2. Структура каталогов

| Путь | Назначение |
|---|---|
| run.py | Точка входа: create_app(), запуск на 0.0.0.0:5000, debug=True |
| app/__init__.py | Фабрика приложения (create_app), регистрация blueprint'ов, обработчики ошибок 400/404/500, db.create_all() |
| app/config.py | Конфигурация: BaseConfig, DevelopmentConfig, ProductionConfig, TestingConfig; сборка DATABASE_URL из env |
| app/extensions.py | Инициализация db, migrate, login_manager |
| app/models.py | SQLAlchemy-модели |
| app/auth.py, app/utils.py, app/puzzle_captcha.py | Вспомогательные модули (аутентификация, утилиты, капча) |
| app/blueprints/ | Маршруты по доменам: auth, admin, clients, materials, notes, operations, production_orders, products, sales_orders, specifications, web, puzzle_captcha |
| app/services/ | Бизнес-логика: auth_service, client_service, material_service, operation_service, product_service, production_order_service, sales_order_service, specification_service, user_service, base_service |
| app/repositories/ | Доступ к БД: base_repository + репозитории по доменам |
| app/dtos/ | DTO: client_dto, material_dto, note_dto, operation_dto, product_dto, production_order_dto, sales_order_dto, specification_dto, user_dto |
| app/views/ | Class-Based Views: admin_view, client_view, material_view, note_view, operation_view, product_view, production_order_view, sales_order_view, specification_view, base_view |
| database/ | schema.sql (схема БД), order_cost_query.sql (запрос расчёта стоимости) |
| tests/ | conftest.py, test_auth.py, test_notes.py |
| web/ | HTML-шаблоны + styles.css |
| image/ | Скриншоты 1.png–4.png |
| docs/ | Документация; сюда добавлены отчёты ревью (docs/review/) |
| scripts/ | Отсутствует — замечание для отчёта |

## 3. Запуск проекта (проверено на macOS, Apple Silicon)

### 3.1. Предварительные требования
- Python 3.13
- PostgreSQL (в ходе ревью использован Homebrew PostgreSQL 19)
- Git, аккаунт GitHub

### 3.2. Клонирование и ветка
git clone https://github.com/<ВАШ_ЛОГИН>/Demo27.git
cd Demo27
git checkout -b review/danilchenkov

### 3.3. Виртуальное окружение и зависимости
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
pip install "psycopg[binary]"      # ВАЖНО: в requirements.txt указан psycopg2-binary, а нужен psycopg v3

### 3.4. База данных
brew services start postgresql@19
psql -d postgres -c "CREATE ROLE postgres WITH LOGIN SUPERUSER PASSWORD '12345';"
psql -U postgres -h localhost -d postgres -c "CREATE DATABASE production_db OWNER postgres;"

Проверка:
psql -U postgres -d production_db -h localhost -c "SELECT current_user, current_database();"
# postgres | production_db

### 3.5. Запуск
python run.py
# → http://127.0.0.1:5000

Если порт 5000 занят (AirPlay Receiver на macOS):
flask --app run run --port 5001
# → http://127.0.0.1:5001

## 4. Замечания, выявленные уже на этапе запуска

| № | Файл | Категория | Приоритет | Замечание |
|---|---|---|---|---|
| 1 | app/config.py | Безопасность | Критично | Пароль БД по умолчанию 12345 захардкожен в _build_database_url(). Должен браться только из env, без дефолта. |
| 2 | app/config.py | Безопасность | Критично | SECRET_KEY имеет дефолт production-is-system-secret-key-change-me — при отсутствии env приложение стартует с известным ключом. |
| 3 | requirements.txt | Зависимости | Важно | Указан psycopg2-binary, но SQLAlchemy 2.x/Flask-SQLAlchemy 3.1 импортируют psycopg (v3). После чистой установки проект не стартует (ModuleNotFoundError: No module named 'psycopg'). |
| 4 | run.py | Логика/Безопасность | Важно | app.run(host='0.0.0.0', debug=True) — сервер слушает все интерфейсы с debug в точке входа. |
| 5 | app/__init__.py | Логика | Важно | db.create_all() вызывается при каждом старте приложения — не заменяет миграции. |
| 6 | app/__init__.py | Логика | Рекомендация | Параметр config_name в create_app() игнорируется. |
| 7 | app/config.py | Логика/Тесты | Важно | TestingConfig использует test_production_db, но тестовая БД нигде не создаётся автоматически. |
| 8 | app/__init__.py | Безопасность | Важно | load_user делает int(user_id) без обработки ValueError. |
| 9 | app/__init__.py | Логика | Рекомендация | check_db_connection() ловит Exception и логирует, но приложение продолжает работу с нерабочей БД. |
| 10 | Корень | Документация | Рекомендация | Отсутствуют README.md, scripts/, docs/ с описанием; нет инструкции по запуску. |
