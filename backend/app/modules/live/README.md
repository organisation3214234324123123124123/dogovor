# live

Живая панель встречи: `WS /ws/meetings/{id}` (корень, не `/api/v1`; прокси на сервере пропускает `/ws`).
Вход только по cookie `access_token` (браузер не ставит заголовки в WebSocket).

Коды закрытия: 4401 — нет сессии или токен истёк, 4403 — Origin не в `cors_origins`, 4404 — чужая или несуществующая встреча, 1011 — сбой транспорта, 1013 — подписчик не успевает.

Сообщения сервера описаны схемой `LiveEvent` в `openapi.yaml` (`item.created`, `item.updated`, `item.deleted`, `gaps.updated`, `closure.opened`, `agenda.updated`) и несут `version` пункта (FR-14, NFR-05). Догрузки пропущенных кадров нет: после переподключения клиент перечитывает `GET /meetings/{id}`.

Кадры шлёт `service.publish_after_commit`: они уходят в шину только после commit и пропадают при откате. Шину выбирает настройка `live_bus` (`memory` или `redis`).
