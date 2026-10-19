# API

Источник: SRS v3, раздел «API», «Письма и ссылки», «Конкурентные правки и живая панель». Целевой контракт со схемами — `docs/api/openapi.yaml`; контракт аналитиков — `openapi-v2.yaml` (код с ним сведён; расхождения, где оставлен наш вариант, — в разделе «Расхождения с контрактом аналитиков» ниже). Восемь правок к таблице SRS приняты 24.10.2026, они перечислены в конце файла.

## Соглашения
- Базовый путь `/api/v1`, формат JSON. Даты — `2026-10-16` (в поясе серии), метки времени — ISO 8601 в UTC.
- Сессия ведущего — JWT в httpOnly-cookie на 7 дней. Swagger генерирует FastAPI, доступен на `/api/docs`.
- Служебный доступ `/admin/*` — отдельная схема безопасности: заголовок `X-Admin-Token`, сравнивается с переменной `ADMIN_TOKEN` (пустая — `/admin` выключен, 403 `ADMIN_DISABLED`). Cookie и bearer ведущего там не действуют (OQ-07).
- Изменение пункта передаёт `If-Match: <version>`; нет заголовка — 428 `PRECONDITION_REQUIRED`, несовпадение — 409 `VERSION_CONFLICT`.
- Каждый запрос ведущего фильтруется по `owner_user_id`. Чужой объект — всегда 404, а не 403.
- Ошибки — RFC 9457 (`application/problem+json`): `{ "type", "title", "status", "code", "errors" }`. Поле `code` стабильно для фронтенда.
- Слои бэкенда: роуты → сервисы → репозитории.

**Таблица кодов — единственный источник правды.** `docs/api/openapi.yaml`, FR и OQ ссылаются на неё; новый код сначала появляется здесь.

| HTTP | code | Когда |
| --- | --- | --- |
| 401 | UNAUTHORIZED | Нет сессии или она истекла; неверный e-mail или пароль при входе |
| 403 | NOT_YOUR_ITEM | Действие по ссылке над чужим пунктом |
| 403 | CSRF_INVALID | POST по ссылке без CSRF-токена или с неверным (FR-32) |
| 403 | ADMIN_FORBIDDEN | `/admin/*` без заголовка `X-Admin-Token` или с неверным; сессия ведущего не подходит |
| 403 | ADMIN_DISABLED | `/admin/*`, а `ADMIN_TOKEN` не задан (служебный доступ выключен) |
| 404 | NOT_FOUND | Объекта нет или он принадлежит другому ведущему |
| 409 | INVALID_TRANSITION | Переход статуса пункта не из таблицы переходов |
| 409 | VERSION_CONFLICT | `If-Match` не совпал с `items.version` |
| 409 | ALREADY_SENT | `closure` или `accept-in-meeting` у встречи в `sent` (OQ-03) |
| 409 | MEETING_NOT_LIVE | Действие панели у встречи не в нужном статусе: закрытие не из `live`; пункт во встречу в `planned` |
| 409 | MEETING_ALREADY_LIVE | «Начать встречу», а в серии уже есть встреча в `live` или `closing` (OQ-05) |
| 409 | SERIES_ARCHIVED | Изменение архивной серии, новая встреча в ней (FR-09) |
| 409 | EMAIL_TAKEN | Регистрация с занятым e-mail, в любом регистре (FR-01) |
| 409 | SUMMARY_NOT_READY | «Скопировать итог» у встречи в `live` (FR-36) |
| 410 | LINK_EXPIRED | Ссылка исполнителя или ссылка сброса пароля истекла, отозвана или использована |
| 422 | VALIDATION_ERROR | Неверные данные запроса |
| 422 | DUE_IN_PAST | Срок раньше сегодня по поясу серии (BR-2) |
| 422 | DUE_TOO_LATE | Исполнитель по ссылке переносит срок позже исходного + 60 дней (BR-6). Ведущего это не ограничивает |
| 422 | USE_REASSIGN | `PATCH /items/{id}` меняет или снимает `assignee_person_id` в pending, accepted, disputed (FR-16.4); исполнителя меняет `reassign` |
| 422 | NOT_A_PARTICIPANT | Исполнитель не из состава серии (BR-1) |
| 422 | GAPS_OPEN | Остались пробелы: рассылка без `confirm_open_gaps`; «Согласовано» при блокирующих пробелах, без due_date или с отсутствующим исполнителем (BR-3, OQ-06) |
| 428 | PRECONDITION_REQUIRED | Изменение пункта без заголовка `If-Match` |
| 429 | RATE_LIMITED | Превышен лимит; заголовок Retry-After |
| 501 | NOT_IMPLEMENTED | Ручка есть в контракте, логика ещё не написана; в `detail` номер FR |
| 500 | INTERNAL_ERROR | Необработанная ошибка сервера |

Прочие ответы фреймворка (405, 415 и т. п.) получают `code` = имя статуса HTTP (`METHOD_NOT_ALLOWED`). Решения по отдельным случаям — OQ-06.

## Ручки
| Метод и путь | Что делает | FR |
| --- | --- | --- |
| POST /auth/register, /auth/login, /auth/logout | Регистрация, вход, выход | FR-01, FR-02 |
| POST /auth/password-reset, /auth/password-reset/confirm | Восстановление пароля | FR-03 |
| GET, POST /series; GET, PATCH /series/{id}; POST /series/{id}/archive | Серии; в ответе `load_warning_threshold` (по умолчанию 5, PATCH принимает значения от 1) и `open_items_count` (pending и accepted) | FR-04, FR-09, FR-26 |
| GET, POST /series/{id}/members; DELETE /series/{id}/members/{personId} | Состав серии | FR-05 |
| POST /series/{id}/meetings | Начать встречу; в ответе встреча и список для сверки | FR-06, FR-42 |
| GET, PATCH /meetings/{id} | Встреча: плановый конец, присутствие | FR-06, FR-07 |
| POST /meetings/{id}/agenda; PATCH /agenda/{id} | Повестка (Should; есть только в полном SRS) | FR-08, FR-27 |
| POST /meetings/{id}/items | Добавить решение строкой; встреча в live, closing или sent (BR-5, OQ-02) | FR-10–FR-12 |
| GET, PATCH, DELETE /items/{id} | Карточка (`can_accept_in_meeting`, `accept_blocked_reason`, у пробела `blocking`: BR-3); правка; удаление в draft; после рассылки смена исполнителя — 422 `USE_REASSIGN` | FR-13, FR-15, FR-16, FR-45 |
| GET /items/{id}/history | Журнал item_events; только ведущий — владелец | FR-45 |
| POST /items/{id}/gaps/{gapId}/dismiss | «Не пробел» | FR-24 |
| POST /items/{id}/accept-in-meeting | «Согласовано на встрече»: тело `{"value": bool}`, false снимает (OQ-10) | FR-25 |
| POST /meetings/{id}/closure | Открыть режим закрытия | FR-22, FR-23 |
| POST /meetings/{id}/send | Разослать итог; тело `{"confirm_open_gaps": bool}` (OQ-03); повтор из sent шлёт только новые draft. Ответ: `meeting_id`, `status`, `summary_sent_at`, `items_total`, `items_with_open_gaps`, `items_unassigned`, `recipients` (число участников встречи) и наши `meeting`, `items_sent`, `notifications_queued`, `already_sent` | FR-28, FR-29 |
| POST /meetings/{id}/summary-text | Текст итога для мессенджера; событие `summary_copied` | FR-36 |
| GET /meetings/{id}/review | Список сверки | FR-42 |
| POST /meetings/{id}/review/skip | Пропустить сверку; событие `review_skipped` | FR-43 |
| POST /items/{id}/actions | done, reschedule, cancel, reassign (из pending, accepted, disputed), reopen (Should, этап 2) | FR-42, FR-43, FR-45 |
| GET /series/{id}/items | История пунктов: фильтры status, assignee_person_id, meeting_id, overdue; поле `on_time_share_30d` (Should) | FR-44 |
| WS /ws/meetings/{id} | События: item.created, item.updated, gaps.updated, closure.opened, agenda.updated, meeting.sent (кадр несёт имя и в `type`, и в `event`) | FR-14 |
| GET /p/{token} | Данные страницы по ссылке (JSON, OQ-01); без изменения состояния; ставит cookie `csrf_token` | FR-31 |
| POST /p/{token}/items/{id}/{accept, reschedule, decline, done} | Действия исполнителя; CSRF-токен обязателен | FR-31, FR-32 |
| GET /admin/export/events.csv | Выгрузка событий за период, параметры `from` и `to` (даты, UTC, включительно): `name,created_at,user_id,series_id,meeting_id,item_id,props` (`X-Admin-Token`) | FR-51 |
| POST /admin/persons/{id}/anonymize | Обезличивание (`X-Admin-Token`) | FR-52 |
| GET, POST /admin/llm/prompts | Версии промптов; POST — новая версия (`X-Admin-Token`, OQ-07) | FR-12 |
| GET /health | Схема `Health`: `status` (ok, degraded, down), `db`, `email_queue` {queued, failed_24h, oldest_queued_age_s}, `llm` {available, provider}, `worker_last_run_at`, `version`. При сбое БД — 503, `db: down` | NFR-11 |

Ручки сверх списка SRS (summary-text, review, review/skip, llm/prompts, `GET /auth/me`, `GET /series/{id}/meetings`) — OQ-07. Ручки этапа 3 (`WS /ws/meetings/{id}/audio`) сюда не входят.

## Страница по ссылке
- Отдельный лёгкий бандл SPA. Заголовки `Referrer-Policy: no-referrer` и `X-Robots-Tag: noindex`; сторонних скриптов и счётчиков нет.
- Действия только POST с CSRF-токеном (double submit): `GET /p/{token}` отдаёт `csrf_token` в теле и ставит cookie `csrf_token` (`HttpOnly; SameSite=Strict; Path=/api/v1/p`); POST шлёт токен в заголовке `X-CSRF-Token`, он должен совпасть с cookie, иначе 403 `CSRF_INVALID`. GET никогда не меняет статус пунктов (BR-12).
- Кнопка из письма открывает страницу с уже выбранным действием и кнопкой подтверждения.
- Чужой пункт — 403 `NOT_YOUR_ITEM`; истёкшая или отозванная ссылка — 410 `LINK_EXPIRED` и «Ссылка устарела» с именем ведущего, без данных пунктов.
- Ограничение: 30 запросов в минуту с IP (nginx `limit_req`).

## Живая панель
- `WS /ws/meetings/{id}`; после обрыва переподключение за 5 с, догрузка — перечитывание `GET /meetings/{id}` (FR-14, NFR-05, OQ-07).

## Принятые правки (24.10.2026)
1. `GET /auth/me` отдаёт текущего ведущего; по нему SPA понимает, что вход выполнен.
2. Текст для «Скопировать итог» (FR-36): простой текст без персональных ссылок. У нас `POST /meetings/{id}/summary-text`, а не GET: ручка пишет событие `summary_copied` и меняет состояние.
3. Страницу по ссылке рисует SPA, данные берёт JSON-ом из `GET /api/v1/p/{token}`; в ответе же CSRF-токен.
4. Версии промптов: список, новая версия, «сделать активной» (`/admin/prompts`, `/admin/prompts/{id}/activate`). Активация — этап 2.
5. В `/admin/*` ходят со служебным ключом в заголовке `X-Admin-Token`.
6. Ручки повестки (этап 2, Should): добавить тему и связать с решениями.
7. Чужой объект в кабинете — 404, а не 403 (NFR-09). 403 остаётся на странице по ссылке: чужой пункт `NOT_YOUR_ITEM`, неверный CSRF `CSRF_INVALID`.
8. Живая панель: события `item.deleted` и `meeting.sent`; `resync` после обрыва у нас заменён перечитыванием `GET /meetings/{id}`.

## Расхождения с контрактом аналитиков
Сведено с `openapi-v2` (SRS v3, правки приняты 24.10). Кадр WS несёт имя события и в `type` (наше), и в `event` (контракт). `/health` при сбое БД отвечает 503 (`status: down`, `db: down`). `load_warning_threshold` по умолчанию 5, значение меньше 1 — 422. `DUE_TOO_LATE` возвращается только при переносе срока исполнителем по ссылке (BR-6). `USE_REASSIGN` — на любое изменение `assignee_person_id` в pending, accepted, disputed. Код `CSRF_FAILED` заменён на `CSRF_INVALID`.

Оставлен наш вариант вместо контракта (вопрос аналитикам): `POST` для summary-text; догрузка WS перечитыванием вместо resync/events_version; 403 вместо 401 на `/admin`; `GET /review`; ответ `/p` на action — `PublicItem`; anonymize отвечает 200 с телом; пагинация limit/offset; `/auth/token`; 428 без If-Match; snake_case и имена схем; обязательный `pd_consent`.
