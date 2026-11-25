# Отчёт по стилю и читаемости кода (День 2)

**Дата:** 2025-11-25
**Ревьюер:** Данильченков Роман
**Ветка:** review/danilchenkov
**Инструменты:** flake8 7.5.0, pylint 4.1.4
**Команды:**
- `flake8 app/ --max-line-length=100 --statistics --count`
- `pylint app/ --disable=C0114,C0115,C0116`

---

## 1. Сводка flake8

Всего нарушений: **142**.

| Код | Описание | Кол-во |
|---|---|---|
| W293 | blank line contains whitespace | 83 |
| E501 | line too long (>100) | 46 |
| F401 | imported but unused | 10 |
| E303 | too many blank lines (3) | 2 |
| W291 | trailing whitespace | 1 |

### Топ файлов-нарушителей (по flake8)

| Файл | Кол-во нарушений | Основные коды |
|---|---|---|
| `app/views/sales_order_view.py` | ~50 | W293, E501, F401 |
| `app/views/product_view.py` | ~8 | W293, E501 |
| `app/views/specification_view.py` | ~6 | W293, E501 |
| `app/views/production_order_view.py` | ~6 | W293, E501 |
| `app/services/sales_order_service.py` | ~8 | W293, F401, E501 |
| `app/services/production_order_service.py` | ~8 | W293, F401, E501 |
| `app/models.py` | 3 | E501 |
| `app/dtos/*.py` | 5 | E501 |

### Типовые примеры

- `app/views/sales_order_view.py:431:101: E501 line too long (197 > 100 characters)` — самая длинная строка в проекте.
- `app/views/sales_order_view.py:230:39: W291 trailing whitespace`
- `app/services/sales_order_service.py:33:9: F401 'app.models.ProductPrice' imported but unused`
- `app/views/material_view.py:139:1: E303 too many blank lines (3)`

---

## 2. Сводка pylint

**Итоговый скор: 8.84/10**

### Системные замечания

| Код | Описание | Где | Кол-во |
|---|---|---|---|
| C0415 | Import outside toplevel (импорт внутри функции) | `app/__init__.py`, `app/views/*`, `app/blueprints/*`, `app/repositories/user_repository.py` | ~25 |
| W0718 | Catching too general exception Exception | `app/__init__.py`, `app/blueprints/notes.py`, `puzzle_captcha.py`, `web.py` | ~6 |
| C0303 | Trailing whitespace | `app/views/*` | много |
| C0301 | Line too long | `app/models.py`, `app/dtos/*`, `app/views/*` | много |
| R0903 | Too few public methods (0/2) | `app/config.py`, `app/models.py` (модели) | ~20 |
| R0914 | Too many local variables | `app/__init__.py`, `app/puzzle_captcha.py`, `app/views/production_order_view.py` | 4 |
| R1705 | Unnecessary "else"/"elif" after "return" | `app/puzzle_captcha.py` | 2 |
| R0902 | Too many instance attributes | `app/dtos/sales_order_dto.py`, `production_order_dto.py` | 2 |
| W0404 | Reimport 'db' | `app/views/production_order_view.py:73` | 1 |
| C0411 | Wrong import order | `app/views/*`, `app/blueprints/puzzle_captcha.py` | ~5 |
| R1731 | Consider using max() | `app/views/production_order_view.py:59` | 1 |

> `R0903 too-few-public-methods` для моделей и конфигов — ложноположительное, игнорируем (это декларативные классы).

---

## 3. Замечания по структуре проекта (задание 2.1)

| № | Файл / область | Замечание | Приоритет |
|---|---|---|---|
| 1 | `app/auth.py` и `app/blueprints/auth.py` | Дублирование модулей аутентификации. Нужно проверить, какой используется, а какой — мёртвый код. | Важно |
| 2 | `app/puzzle_captcha.py` и `app/blueprints/puzzle_captcha.py` | Аналогичное дублирование: логика капчи в двух местах. | Важно |
| 3 | `app/blueprints/web.py` и `app/views/*` | Смешение двух подходов: function-based blueprints (`blueprints/`) и Class-Based Views (`views/`). Нет единого стандарта. | Рекомендация |
| 4 | `image/` в корне | Скриншоты `1.png`–`4.png` лежат в корне репозитория. Лучше перенести в `docs/images/` или удалить. | Рекомендация |
| 5 | `scripts/` | Каталог отсутствует, хотя в задании упомянут. Обосновать отсутствие или добавить скрипты запуска/инициализации БД. | Рекомендация |
| 6 | `app/config.py` | 4 класса конфигурации с `R0903 too-few-public-methods` — это нормально для конфигов, но стоит вынести `_build_database_url` в отдельный модуль утилит. | Рекомендация |
| 7 | `app/dtos/*` | DTO-классы с `R0902 too-many-instance-attributes` (8–10 полей). Рассмотреть разбиение или использование `dataclass`. | Рекомендация |

---

## 4. Замечания по докстрингам и комментариям (задание 2.3)

| № | Файл | Замечание | Приоритет |
|---|---|---|---|
| 1 | `app/__init__.py`, `app/config.py` | Докстринги есть, но **избыточно многословные** — комментарии «для студентов» (обращения «Студенты: …»), объясняют базовые вещи. В продакшн-коде это шум. | Рекомендация |
| 2 | `app/views/*`, `app/blueprints/*` | Во многих методах отсутствуют докстринги (pylint `C0116` отключён, но по факту их нет). | Важно |
| 3 | `app/services/*`, `app/repositories/*` | Нужно проверить наличие докстрингов у публичных методов (в отчёт не вошло из-за `--disable=C0116`). | Важно |
| 4 | `app/__init__.py` | Комментарии вида «Почему используется паттерн…» полезны для обучения, но дублируют документацию. Лучше вынести в `docs/`. | Рекомендация |
| 5 | `app/puzzle_captcha.py` | Нет модульного докстринга, объясняющего назначение капчи и формат данных. | Важно |

---

## 5. Замечания по читаемости кода

| № | Файл / строка | Замечание | Приоритет |
|---|---|---|---|
| 1 | `app/views/sales_order_view.py:431` | Строка длиной **197 символов** — читаемость резко падает. | Важно |
| 2 | `app/views/sales_order_view.py:416` | Строка 160 символов. | Важно |
| 3 | `app/views/sales_order_view.py:382` | Строка 140 символов. | Важно |
| 4 | `app/views/*` (много) | Строки с пробелами в конце (`W293`, `W291`) — шум в diff, лучше настроить автоформатирование. | Рекомендация |
| 5 | `app/views/production_order_view.py:59` | Ручная проверка `if num > max_num: max_num = num` — pylint предлагает `max()`. | Рекомендация |
| 6 | `app/views/production_order_view.py:73` | Повторный импорт `db` внутри функции (уже импортирован на строке 27). | Важно |
| 7 | `app/__init__.py` | 20 локальных переменных в `create_app()` (порог 15) — функция перегружена, стоит разбить на `_register_blueprints(app)`, `_register_error_handlers(app)`, `_init_db(app)`. | Важно |
| 8 | `app/blueprints/web.py` | 7 `return` в одной функции (`R0911`) — сложно читать, разбить на хелперы. | Рекомендация |
| 9 | `app/puzzle_captcha.py` | `else`/`elif` после `return` — лишний уровень вложенности. | Рекомендация |

---

## 6. Рекомендации по автоматизации

1. Добавить в проект `setup.cfg` или `.flake8` с настройками:
2. Подключить `black` + `isort` для автоформатирования (устранит 90% W293/E501/W291).
3. Добавить `pre-commit`-хук с flake8, чтобы не коммитить нарушения.
4. Включить в CI (GitHub Actions) шаг `flake8 app/ --max-line-length=100`.

---

## 7. Выводы

- Код в целом читаемый, pylint 8.84/10 — выше среднего.
- Основная масса нарушений flake8 — **косметические** (пробелы, длина строк), устраняются автоформатированием.
- Системные проблемы: **импорты внутри функций** (`C0415`), **широкие `except Exception`** (`W0718`), **дублирование модулей** (`auth`, `puzzle_captcha`).
- Докстринги есть не везде; в `app/__init__.py` и `app/config.py` — избыточны и «учебные».
- Рекомендуется внедрить `black`/`isort`/`pre-commit` и разбить `create_app()` на хелперы.