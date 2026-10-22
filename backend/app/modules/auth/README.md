# auth

Вход ведущего. Сессия — JWT HS256 (подпись `SECRET_KEY`, 7 дней) в httpOnly-cookie `access_token`;
тот же токен в теле ответа для Swagger «Authorize» и скриптов (`Authorization: Bearer`).
`CurrentUser` (`core/security.py`) читает cookie, затем bearer; нет или истёк — 401 `UNAUTHORIZED`.

| Ручка | FR | Статус |
| --- | --- | --- |
| `POST /auth/login` (JSON или форма OAuth2) | FR-01 | работает |
| `POST /auth/logout` | FR-01 | работает |
| `GET /auth/me` | FR-01 | работает (дополнение к SRS) |
| `POST /auth/register` | FR-01, NFR-12 | работает: `pd_consent_at`, своя сторона, 409 `EMAIL_TAKEN` |
| `POST /auth/password-reset`, `/auth/password-reset/confirm` | FR-03 | работает: токен 43 символа, в БД SHA-256, 1 час, один раз, 410 `LINK_EXPIRED` |

