# items

Пункты встречи и карточка пункта. Просрочка (`is_overdue`) вычисляется по поясу серии. Изменение пункта требует `If-Match: <version>` (ETag из `GET /items/{id}`): без заголовка 428, при несовпадении 409 `VERSION_CONFLICT`.

| Ручка | FR |
| --- | --- |
| `GET /items/{id}` (карточка, последний ответ исполнителя) | FR-45 |
| `GET /items/{id}/history` | FR-45 |
| `GET /series/{id}/items` (фильтры status, assignee_person_id, meeting_id, overdue; `on_time_share_30d`) | FR-44 |
| `POST /meetings/{id}/items` | FR-10, FR-11 (FR-12 — точка `ENRICHERS`) |
| `PATCH /items/{id}` | FR-13, FR-16, FR-26 |
| `DELETE /items/{id}` | FR-15 |
| `POST /items/{id}/gaps/{gapId}/dismiss` | FR-24 |
| `POST /items/{id}/accept-in-meeting` (`{"value": bool}`) | FR-25, OQ-10 |
| `POST /items/{id}/actions` (done, reschedule, cancel, reassign, reopen) | FR-43, FR-45 |

Строку с решением разбирает `parsing.parse_line` — чистая функция без сети и БД. В ответ на создание пункта и смену исполнителя добавляется предупреждение о загрузке (FR-26). Раз в 5 минут Beat пишет события `item_overdue` для серий, у которых наступило 00:05 по их поясу (FR-41).

Другим модулям доступны `service.to_reads`, `service.list_meeting_items`, `service.today_in`.
