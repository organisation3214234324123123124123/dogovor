# series

Серии, состав, встречи, повестка, обезличивание участника. Все запросы фильтруются по
`owner_user_id`; чужое — 404.

| Ручка | FR |
| --- | --- |
| `GET /series`, `GET /series/{id}` | FR-04 |
| `GET /series/{id}/members` | FR-05 |
| `GET /series/{id}/meetings` | — (дополнение к SRS) |
| `GET /meetings/{id}` (панель: участники, повестка, пункты, пробелы) | FR-06, FR-14 |
| `POST /series`, `PATCH /series/{id}` | FR-04 |
| `POST /series/{id}/archive`, `DELETE /series/{id}/members/{personId}` | FR-09 |
| `POST /series/{id}/members` | FR-05 |
| `POST /series/{id}/meetings` (встреча + сверка) | FR-06, FR-42 |
| `PATCH /meetings/{id}` | FR-06, FR-07 |
| `POST /meetings/{id}/closure` (+ Beat `open_closures` за 5 минут до конца) | FR-22, FR-23, FR-27 |
| `POST /meetings/{id}/send` (`confirm_open_gaps`) | FR-28, FR-29 |
| `POST /meetings/{id}/summary-text` | FR-36 |
| `GET /meetings/{id}/review`, `POST /meetings/{id}/review/skip` | FR-42, FR-43 |
| `POST /meetings/{id}/agenda`, `PATCH /agenda/{id}` | FR-08, FR-27 |
| `POST /admin/persons/{id}/anonymize` (`X-Admin-Token`) | FR-52 |

Другим модулям доступны `service.get_owned_series` и `service.get_owned_meeting`.

Сверка (FR-42) собирается в `items.service.review_groups`; порядок групп и правила — в [docs/decisions.md](../../../../docs/decisions.md). Старт встречи блокирует серию (гонка даёт 409 `MEETING_ALREADY_LIVE`), добавление в повестку блокирует встречу.
