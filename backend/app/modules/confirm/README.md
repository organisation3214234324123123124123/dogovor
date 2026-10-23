# confirm

Страница исполнителя по ссылке из письма, без входа. Токен случайный, в базе только его SHA-256; ответы идут с `Referrer-Policy: no-referrer` и `X-Robots-Tag: noindex`. GET ничего не меняет (BR-12), а POST защищён double submit CSRF (FR-32): заголовок `X-CSRF-Token` должен совпасть с cookie `csrf_token`, которую ставит GET.

| Ручка | FR |
| --- | --- |
| `GET /p/{token}` | FR-31, FR-35 |
| `POST /p/{token}/items/{id}/accept` | FR-31 |
| `POST /p/{token}/items/{id}/reschedule` | FR-31, BR-6 |
| `POST /p/{token}/items/{id}/decline` | FR-31 |
| `POST /p/{token}/items/{id}/done` | FR-31 |

Неизвестная ссылка даёт 404, истёкшая — 410 `LINK_EXPIRED` с именем ведущего (`host_name`).

Ссылка одна на пару (человек, встреча), истёкшую сервис сначала отзывает (FR-30). Каждое письмо получает свой токен в `access_link_tokens`; все они гаснут вместе со ссылкой. Ответ исполнителя записывает `items.service.answer_by_link`.

Демо-токены: `demo-ivan`, `demo-anna`, `demo-oleg`, `demo-kate`.
