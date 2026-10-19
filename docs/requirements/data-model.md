# Модель данных

Источник: SRS v3, разделы «Модель данных», «Жизненный цикл пункта», «Поиск пробелов», «Письма и ссылки», «Время».
Метки времени — `timestamptz` в UTC; сроки — `date` без времени в поясе серии; e-mail — `citext`. Тип id в SRS не задан (ER-диаграмма допускает uuid).
Текст разговора и звук в базе не хранятся (BR-14), сроки хранения — BR-20. Всего 19 таблиц; `tracker_connections`, `tracker_links` — этап 4, сейчас не нужны.

## Таблицы
| Таблица | Поля | Ключи и ограничения |
| --- | --- | --- |
| users | id, email, password_hash, full_name varchar(200), timezone = 'Europe/Moscow', pd_consent_at timestamptz null, created_at | email UNIQUE. В MVP только ведущие. pd_consent_at — согласие на обработку ПДн (NFR-12, OQ-11), пишется при регистрации |
| organizations | id, owner_user_id, name varchar(200), is_own bool = false, created_at | UNIQUE(owner_user_id, lower(name)); у ведущего ровно одна is_own = true |
| persons | id, owner_user_id, organization_id, full_name varchar(200), email, anonymized_at, created_at | UNIQUE(owner_user_id, email); organization_id → organizations |
| series | id, owner_user_id, title varchar(200), timezone, planned_duration_min smallint, capture_mode enum(manual, audio) = manual, status enum(active, archived), load_warning_threshold int null, archived_at timestamptz null (BR-20), created_at | INDEX(owner_user_id, status); длительность 15–240 (FR-04); load_warning_threshold ≥ 1, NULL — порог по умолчанию 5 (BR-9, OQ-11) |
| series_members | series_id, person_id | PK(series_id, person_id) |
| meetings | id, series_id, planned_start, planned_end, status enum(planned, live, closing, sent), started_at, closure_opened_at, closure_trigger enum(button, timer), summary_sent_at, created_at | INDEX(series_id, planned_start DESC); в серии не больше одной встречи в live или closing |
| meeting_participants | meeting_id, person_id, present bool = true | PK(meeting_id, person_id) |
| agenda_items (Should) | id, meeting_id, position smallint, title varchar(300), status enum(open, covered, moved) | UNIQUE(meeting_id, position); до 20 пунктов (FR-08) |
| items | id, meeting_id, series_id, agenda_item_id, raw_text varchar(1000), what varchar(500), assignee_person_id, due_date, due_text varchar(100), done_criteria varchar(500), manual_fields text[], source enum(manual, asr), suggested bool = false, status enum(draft, pending, accepted, disputed, done, cancelled), accepted_in_meeting bool = false, accepted_at, sent_at, done_at, done_by enum(assignee, host, tracker), cancelled_at, reschedule_count int = 0, version int = 1, created_at, updated_at | INDEX(series_id, status); INDEX(assignee_person_id); частичный INDEX(due_date) WHERE status IN (pending, accepted); version — оптимистичная блокировка |
| gaps | id, item_id, type enum(no_assignee, no_due, due_vague, due_past, vague_action), source enum(rule, llm), question varchar(300), detected_at, resolved_at, dismissed_at | INDEX(item_id) WHERE resolved_at IS NULL AND dismissed_at IS NULL |
| access_links | id, token_hash bytea(32), person_id, meeting_id, created_at, expires_at, revoked_at, last_used_at | token_hash UNIQUE; одна действующая ссылка на пару (person_id, meeting_id) |
| notifications | id, type enum(SUMMARY, CONFIRM_REQUEST, CONFIRM_REPEAT, HOST_RESPONSE, DUE_TOMORROW, ITEM_CHANGED, ITEM_REASSIGNED, PASSWORD_RESET), recipient_person_id, recipient_user_id, meeting_id, item_id, payload jsonb, dedupe_key, scheduled_for, status enum(queued, sent, failed, cancelled), attempts smallint = 0, last_error, sent_at | dedupe_key UNIQUE; INDEX(status, scheduled_for); ровно один из recipient_* заполнен. ITEM_CHANGED и ITEM_REASSIGNED в таблице SRS не добавлены, но описаны в разделе писем; PASSWORD_RESET — для FR-03 (OQ-11). Форматы dedupe_key — [BR.md](BR.md#dedupe_key-br-5) |
| item_events | id bigserial, item_id, actor_type enum(host, assignee, system, tracker), actor_user_id, actor_person_id, action enum, data jsonb, created_at | Только INSERT: UPDATE и прямой DELETE запрещены триггером; строки уходят только каскадом вместе с пунктом — удаление draft (FR-15) и чистка BR-20: пункты уходят каскадом, события старше года — прямым DELETE при `app.retention_cleanup = on` (OQ-17). В data только id, статусы и даты (BR-19) |
| item_comments | id, item_id, author_person_id, kind enum(not_mine, reschedule), text varchar(500), created_at, anonymized_at | INDEX(item_id). Свободный текст исполнителя живёт только здесь |
| analytics_events | id bigserial, name, user_id, series_id, meeting_id, item_id, props jsonb, created_at | INDEX(name, created_at); в props нет текста пунктов |
| llm_prompts | id, task enum(parse, gaps, question), version int, text, is_active bool = false, created_at, created_by | UNIQUE(task, version); частичный UNIQUE(task) WHERE is_active (на обсуждении, OQ-11). Стартовые версии миграцией, дальше только новые строки |
| llm_calls | id, meeting_id, task enum(parse, gaps, question), provider, model, prompt_id, latency_ms, input_tokens, output_tokens, outcome enum(ok, timeout, invalid_output), created_at | Без текста запроса и ответа. prompt_id → llm_prompts |

Связи: ведущий владеет организациями, людьми и сериями; серия → встречи; у встречи участники, повестка, пункты; у пункта пробелы, журнал, комментарии. Пункт привязан к встрече, где родился, и к серии.

## Статусы пункта и переходы
`draft` живёт только до рассылки. Просрочка — не статус: `is_overdue = due_date < сегодня (пояс серии) AND status IN (pending, accepted)`.

| Из | В | Действие | Кто | Побочные эффекты |
| --- | --- | --- | --- | --- |
| draft | accepted | «Разослать», отмечен «Согласовано на встрече» | Ведущий | sent_at, accepted_at; SUMMARY со ссылкой; DUE_TOMORROW (BR-8) |
| draft | pending | «Разослать», без отметки | Ведущий | sent_at; ссылка; SUMMARY и CONFIRM_REQUEST; CONFIRM_REPEAT (BR-10); DUE_TOMORROW. Без исполнителя — pending без ссылки (BR-4) |
| draft | — | Удалить | Ведущий | Физическое удаление вместе с пробелами |
| pending | accepted | «Принимаю» | Исполнитель | accepted_at; CONFIRM_REPEAT отменяется |
| pending, accepted | accepted | «Нужен другой срок» | Исполнитель | Новый due_date (BR-6); reschedule_count + 1; accepted_at если пусто; комментарий в item_comments; HOST_RESPONSE; DUE_TOMORROW перепланируется; срок ссылки пересчитывается (BR-18) |
| pending, accepted | disputed | «Это не моё» | Исполнитель | Комментарий до 500 символов, необязателен, в item_comments; HOST_RESPONSE |
| pending, accepted, disputed | pending | «Переназначить» | Ведущий | Новый исполнитель: ссылка и CONFIRM_REQUEST; прежнему ITEM_REASSIGNED; прежняя ссылка не показывает пункт |
| pending, accepted | done | «Сделано» | Исполнитель (только из accepted), ведущий, трекер (этап 4) | done_at, done_by; напоминание отменяется |
| pending, accepted | тот же | «Перенести» (сверка, карточка) | Ведущий | Новый due_date; reschedule_count + 1; напоминание перепланируется; ITEM_CHANGED |
| pending, accepted, disputed | тот же | Правка «что», срока, «готово» (FR-16) | Ведущий | version + 1; при смене срока reschedule_count + 1 и перепланирование; ITEM_CHANGED |
| pending, accepted, disputed | cancelled | «Снять» | Ведущий | cancelled_at; напоминание отменяется; писем нет |
| done | accepted | «Вернуть в работу» (Should, этап 2) | Ведущий | done_at = NULL; ITEM_CHANGED |

- Любой другой переход → 409 `INVALID_TRANSITION`. Каждый переход пишет `item_events` и событие аналитики в той же транзакции.
- `reschedule_count` растёт только при смене срока после рассылки (AC-2.4).

## Пробелы
Типы и вопросы по умолчанию — [FR-20](FR-20.md). Готовый вопрос до 120 символов; «Не пробел» ставит dismissed_at (FR-24).

## Письма (notifications)
| Тип | Кому | Когда | dedupe_key |
| --- | --- | --- | --- |
| SUMMARY | Всем участникам встречи | «Разослать», в течение 5 минут | `{meeting_id}:SUMMARY:{person_id}` |
| CONFIRM_REQUEST | Исполнителю | С итогом для pending; переназначение; пункт после рассылки | `{item_id}:CONFIRM_REQUEST:{person_id}:{version}` |
| CONFIRM_REPEAT | Исполнителю с pending | Через 1 рабочий день, один раз | `{meeting_id}:CONFIRM_REPEAT:{person_id}` |
| HOST_RESPONSE | Ведущему | Сразу после «Другой срок» или «Не моё» | `{item_id}:HOST_RESPONSE:{item_event_id}` |
| DUE_TOMORROW | Исполнителю | 09:00 пояса серии за 1 рабочий день до срока | `{item_id}:DUE_TOMORROW:{due_date}:{version}` |
| ITEM_CHANGED | Исполнителю | Правка после рассылки, «Перенести», «Вернуть в работу» | `{item_id}:ITEM_CHANGED:{version}` |
| ITEM_REASSIGNED | Прежнему исполнителю | «Переназначить»; без ссылки и имени нового | `{item_id}:ITEM_REASSIGNED:{person_id}:{version}` |
| PASSWORD_RESET | Ведущему | Запрос сброса пароля (FR-03) | `{user_id}:PASSWORD_RESET:{reset_id}` |

Различители в ключах и почему — [BR.md](BR.md#dedupe_key-br-5), OQ-16.

`scheduled_for` считает бэкенд в UTC по поясу серии; Celery Beat регулярно отправляет наступившие. Повторы — BR-17; объединение — BR-16; лимит — BR-11.

## Токен ссылки
32 байта CSPRNG, base64url; в базе только SHA-256; сравнение за постоянное время; одна действующая ссылка на (человек, встреча); истёкшую, но не отозванную действующую ссылку сервис отзывает перед выдачей новой (FR-30); срок — BR-18; при обезличивании все ссылки отзываются.

## Время
«Сегодня» для BR-2, BR-6, due_past, is_overdue — по поясу серии. Рабочие дни — BR-7. `item_overdue` — ежедневно в 00:05 по поясу серии. Пояс по умолчанию — Europe/Moscow.
