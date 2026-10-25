# auth

Вход ведущего. Сессия — JWT в httpOnly-cookie `session` на 7 дней; `CurrentUser` (`core/security.py`) читает cookie, нет или истёк — 401 `UNAUTHORIZED`. `/auth/token` и `Authorization: Bearer` нужны только для кнопки Authorize в Swagger и работают в режиме разработки (иначе 404 `NOT_FOUND`).

| Ручка | FR |
| --- | --- |
| `POST /auth/login` (JSON; форма OAuth2 — `/auth/token`) | FR-01 |
| `POST /auth/logout` | FR-01 |
| `GET /auth/me` (дополнение к SRS) | FR-01 |
| `POST /auth/register` | FR-01, NFR-12 |
| `POST /auth/password-reset`, `/auth/password-reset/confirm` | FR-03 |

Регистрация сохраняет согласие на обработку ПДн (`pd_consent_at`), создаёт «свою» сторону и отвечает 409 `EMAIL_TAKEN` на занятый e-mail. Токен сброса пароля одноразовый, действует час, просроченный даёт 410 `LINK_EXPIRED`.

FR-02: 10 неудачных входов за 15 минут по одному e-mail (в том числе неизвестному) дают 429 `RATE_LIMITED` с `Retry-After`; попытки хранятся в `login_attempts`, успешный вход их стирает.

