# Assetrix — Project Context for Claude

## Что это
Self-hosted PWA для учёта IT-активов (часть экосистемы Mycorium). Инструмент для IT-техников/сисадминов: быстрый учёт оборудования через QR/штрих-сканирование и пакетные операции. **Не enterprise-система** — приоритет скорости и простоты.

Ключевая метрика: **любая операция ≤ 3–5 секунд со смартфона**.

## Source of truth
- **`01_sources/comp/assetrix_mvp_spec.md`** — приоритетный источник спеки (Design & Scope MVP).
- `01_sources/raw/mvp_info.md` — дополнительная детализация; при расхождении со спекой проигрывает.
- `README.md` — обзор проекта и карта папок; `Backlog.md` — задачи и открытые вопросы.

## Архитектура

### Стек
| Слой | Технология |
|---|---|
| Frontend | React PWA (mobile-first) |
| Backend | FastAPI (Python) |
| Database | PostgreSQL (JSONB для event.meta) |
| Reverse proxy | Caddy (авто-HTTPS) |
| Deploy | Docker (`docker-compose up`) |

### Принцип Event Sourcing (STRICT)
- Все изменения = события в `events`. NO update, NO delete событий.
- API должен возвращать 405/403 на попытку удалить/изменить событие.
- Текущее состояние = derived из последнего события + кешируется в `devices`.
- Должно быть действие «recalculate state» (пересчёт кеша из лога).

## Модель данных

### Device
```
id (UUID), asset_tag (не уникальный), name, type_id (FK),
serial_number (required), created_at
--- кеш ---
physical_condition (enum), operational_status (enum), last_event_id (FK)
```

### DeviceType
```
id, name (unique) — пользовательский плоский справочник, без иерархии
```

### Event (immutable)
```
id, device_id, event_type, note (optional), meta (JSONB), created_at, created_by (nullable)
```

## Dual Status Model (критично)
Каждое устройство имеет **два независимых вектора** состояния:

| Physical condition | Operational status |
|---|---|
| virtual, active, maintenance, damaged, teardown, scrap, disposed | available, assigned, reserved, transit, lost, write_off |

Комбинации валидны: `assigned + damaged`, `write_off + scrap`, `write_off + disposed` и т.д.

## События MVP
`CREATED`, `ASSIGNED`, `RETURNED`, `SENT_TO_REPAIR`, `RETURNED_FROM_REPAIR`, `WRITTEN_OFF`

Бизнес-логика должна блокировать недопустимые переходы (например, `written_off → assign`).

## Scan Sessions (core feature)
Пакетный режим:
1. Выбрать режим (ASSIGN / RETURN / SEND_TO_REPAIR / RETURN_FROM_REPAIR)
2. Задать контекст **до** сканирования (напр. `{"assignee": "text"}`)
3. Сканировать группу (до 50 устройств)
4. Одна кнопка «Применить» — применяет действие ко всем

Дубль-скан уникального устройства → игнорировать.

## Сканирование
- QR содержит `asset_tag`
- Штрих-код — для `serial_number` / vendor codes
- Поддержка: камера смартфона (primary), веб-камера, USB-сканер (keyboard input)
- Неизвестный скан → опции: создать / игнорировать / в pending

## UX-правила
- Mobile-first, управление одной рукой
- Кнопки внизу экрана (зона большого пальца)
- ≤ 3 клика до любого стандартного действия
- Звуковой + визуальный фидбек на каждый скан
- Крупные touch-элементы, минимум текста

## Auth (MVP)
- Один пользователь, авто-логин, без logout
- Поле `created_by` **обязательно в схеме** (под будущее multi-user)

## Performance targets
- До 10 000 устройств
- 2–3 скана/сек (USB штрих-сканер), 1 скан/5 сек (камера)
- Batch до 50 устройств

## Out of Scope (MVP) — не реализовывать
- Мультипользовательский режим, роли
- AD / LDAP интеграция
- Закупки, склад, расходные материалы
- Полные инвентаризации
- Оффлайн-режим

Архитектура должна оставаться **расширяемой** под эти фичи в будущем.

## Definition of Done
1. `docker-compose up` разворачивает систему
2. Открывается на смартфоне по локальному IP, камера читает QR
3. Создано 10 устройств, проведена пакетная ASSIGN
4. События корректно зафиксированы в БД
5. Кеш в `devices` соответствует последнему событию
6. API не даёт изменить/удалить событие (405/403)

## Статус проекта (2026-04-27)
- Код ещё не написан; папок `backend/` и `frontend/` в репозитории нет
- Dev-окружение: 4 Linux-сервера, настраиваются через Ansible с Linux-лаптопа. Настройка запланирована на 2026-04-28.
- Открытые вопросы перед стартом кода: менеджер зависимостей Python, версия Python, QR-библиотека для браузера, формат asset_tag.

## Working Conventions

- Numbered folders hold materials; application code lives in its own folders at the root
- Record tasks and known defects in `Backlog.md`
- Move obsolete files to `06_archive/` instead of deleting them
- New file and folder names: lowercase, no spaces, `_`/`-` separators
