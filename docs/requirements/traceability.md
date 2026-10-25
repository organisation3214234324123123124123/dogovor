# Трассировка требований

Какое требование где в коде и какими тестами проверено. Тесты: `@pytest.mark.fr("FR-xx")` или имя `test_fr_XX*` в `backend/tests/`.
Статусы: `готово` · `частично` · `в работе` · `не начато` · `отложено (причина)`.

| FR | Требование | Приоритет | Этап | Код | Тесты | Статус |
| --- | --- | --- | --- | --- | --- | --- |
| [FR-01](FR-01.md) | Регистрация, вход и выход ведущего | Must | 1 | modules/auth | test_api_auth.py | готово |
| [FR-02](FR-02.md) | Блокировка входа после 10 неудачных попыток | Must | 2, последней | modules/auth | test_fr_02_03_auth_limits.py, test_fr_02_login_lockout.py | готово |
| [FR-03](FR-03.md) | Восстановление пароля | Must | 2, последней | modules/auth | test_fr_02_03_auth_limits.py, test_fr_03_password_reset.py | готово |
| [FR-04](FR-04.md) | Создание серии | Must | 1 | modules/series | test_fr_04_series_create.py, test_api_access.py, test_api_reads.py | готово |
| [FR-05](FR-05.md) | Состав серии | Must | 1 | modules/series | test_fr_05_members.py, test_api_reads.py | готово |
| [FR-06](FR-06.md) | Начать встречу | Must | 1 | modules/series | test_fr_06_start_meeting.py, test_api_reads.py | готово |
| [FR-07](FR-07.md) | Присутствие на встрече | Must | 1 | modules/series | test_fr_07_presence.py | готово |
| [FR-08](FR-08.md) | Повестка встречи | Should | 2 | modules/series | test_fr_08_agenda.py | готово |
| [FR-09](FR-09.md) | Архивирование серии, исключение участника | Should | 2 | modules/series | test_fr_09_archive.py | готово |
| [FR-10](FR-10.md) | Ввод решения строкой | Must | 1 | modules/items | test_fr_10_guards.py, test_fr_10_item_capture.py, test_items_edge_cases.py | готово |
| [FR-11](FR-11.md) | Разбор строки по правилам | Must | 1 | modules/items/parsing.py | test_fr_11_parse_line.py, test_items_edge_cases.py | готово |
| [FR-12](FR-12.md) | Разбор строки нейросетью (LLM) | Must | 2 | modules/llm (сервис) | test_api_admin.py, test_llm_calls.py, test_llm_client.py, test_llm_prompts.py, test_llm_service.py | частично: сервис и заглушка готовы, к `POST /meetings/{id}/items` обогащение через LLM не подключено |
| [FR-13](FR-13.md) | Правка полей пункта в панели | Must | 1 | modules/items | test_fr_13_guards.py, test_fr_13_item_edit.py | готово |
| [FR-14](FR-14.md) | Живая панель по WebSocket | Must | 1 | modules/live | test_items_edge_cases.py, test_api_ws.py | готово |
| [FR-15](FR-15.md) | Удаление пункта в draft | Must | 1 | modules/items | test_fr_15_guards.py, test_fr_15_item_delete.py, test_items_edge_cases.py | готово |
| [FR-16](FR-16.md) | Правка пункта после рассылки | Must | 1 | modules/items, modules/notifications | test_fr_16_edit_after_send.py, test_items_edge_cases.py | готово |
| [FR-20](FR-20.md) | Пробелы по правилам | Must | 1 | modules/gaps | test_fr_20_gap_rules.py, test_fr_20_gaps_private.py | готово |
| [FR-21](FR-21.md) | Размытые формулировки и готовый вопрос (LLM) | Must | 2 | modules/llm (сервис), modules/items | test_llm_service.py | частично: сервис готов, к `POST /meetings/{id}/items` обогащение через LLM не подключено |
| [FR-22](FR-22.md) | Режим закрытия | Must | 1 | — | test_fr_22_23_closure.py, test_fr_50_props_no_text.py | готово |
| [FR-23](FR-23.md) | Автопереход в режим закрытия по таймеру | Should | 2 | — | test_fr_22_23_closure.py | готово |
| [FR-24](FR-24.md) | «Не пробел» | Must | 1 | modules/gaps, modules/items | test_fr_24_dismiss_gap.py, test_fr_24_guards.py, test_items_edge_cases.py | готово |
| [FR-25](FR-25.md) | «Согласовано на встрече» | Must | 1 | modules/items | test_fr_25_accept_in_meeting.py, test_items_edge_cases.py | готово |
| [FR-26](FR-26.md) | Предупреждение о загрузке исполнителя | Should | 2 | modules/items, modules/series | test_fr_26_overload.py, test_items_edge_cases.py | готово |
| [FR-27](FR-27.md) | Контроль повестки в режиме закрытия | Should | 2 | modules/series | test_fr_27_agenda_control.py, test_fr_22_23_closure.py, test_agenda_link_and_sent_count.py | готово |
| [FR-28](FR-28.md) | «Разослать» итог | Must | 1 | — | test_fr_28_send.py, test_agenda_link_and_sent_count.py, test_send_edge_cases.py, test_fr_50_props_no_text.py | готово |
| [FR-29](FR-29.md) | Встреча после рассылки | Must | 1 | — | test_fr_28_send.py | готово |
| [FR-30](FR-30.md) | Письмо CONFIRM_REQUEST с персональной ссылкой | Must | 1 | — | test_fr_30_mails.py, test_fr_28_send.py, test_send_edge_cases.py | готово (п.4: случайный токен на каждое письмо, в БД только SHA-256, хеш фиксируется до SMTP) |
| [FR-31](FR-31.md) | Страница по ссылке и действия исполнителя | Must | 1 | modules/confirm (GET) | test_api_public_link.py, test_fr_31_link_actions.py, test_fr_09_archive_readonly.py | готово (после guard архива) |
| [FR-32](FR-32.md) | Действия по ссылке только POST с CSRF | Must | 1 | modules/confirm | test_api_public_link.py | готово |
| [FR-33](FR-33.md) | Повторное письмо CONFIRM_REPEAT | Must | 1 | — | test_fr_30_mails.py, test_fr_31_link_actions.py | готово |
| [FR-34](FR-34.md) | Письмо ведущему HOST_RESPONSE | Must | 1 | — | test_fr_31_link_actions.py, test_fr_30_mails.py | готово |
| [FR-35](FR-35.md) | Ссылка у каждого исполнителя, включая согласованные пункты | Must | 1 | — | test_fr_30_mails.py, test_fr_31_link_actions.py, test_send_edge_cases.py | готово |
| [FR-36](FR-36.md) | «Скопировать итог» | Must | 1 | — | test_fr_28_send.py, test_send_edge_cases.py, test_fr_50_props_no_text.py | готово |
| [FR-40](FR-40.md) | Напоминание DUE_TOMORROW | Must | 1 | — | test_fr_30_mails.py, test_fr_28_send.py, test_fr_31_link_actions.py | готово |
| [FR-41](FR-41.md) | Просроченный пункт попадает в сверку | Must | 1 | modules/items | test_fr_41_overdue.py | готово |
| [FR-42](FR-42.md) | Сверка в начале встречи | Must | 1 | modules/series, modules/items | test_fr_42_review.py, test_business_days.py | готово |
| [FR-43](FR-43.md) | Действия на сверке | Must | 1 | modules/series, modules/items | test_fr_43_review_actions.py | готово |
| [FR-44](FR-44.md) | Страница серии: история пунктов | Should | 2 | modules/items | test_fr_44_45_history_and_card.py, test_api_reads.py | готово |
| [FR-45](FR-45.md) | Карточка пункта | Must | 1 | modules/items | test_fr_44_45_history_and_card.py, test_api_reads.py | готово |
| [FR-50](FR-50.md) | Запись событий аналитики | Must | 1 | modules/analytics | test_fr_50_analytics_track.py, test_fr_50_rollback.py | готово |
| [FR-51](FR-51.md) | Выгрузка событий и метрик в CSV | Must | 1 | modules/analytics | test_fr_51_export.py | готово |
| [FR-52](FR-52.md) | Обезличивание участника | Must | 1 | modules/analytics, modules/series | test_br_20_retention.py, test_fr_52_anonymize.py, test_committed_sessions.py | готово |

## Правила без своего FR

| Правило | Что сделать | Этап | Код | Тесты | Статус |
| --- | --- | --- | --- | --- | --- |
| BR-20 | Ежедневная чистка по срокам хранения (Celery Beat) | 1 | modules/analytics/tasks.py | test_br_20_retention.py | готово |
| BR-11, BR-16, BR-17 | Лимит, объединение и повторы писем | 1 | modules/notifications | test_fr_16_edit_after_send.py, test_fr_30_mails.py, test_agenda_link_and_sent_count.py | готово |
| NFR-11 | `GET /health` и JSON-логи с request_id | 1 | modules/health, core/logging.py | test_health.py, test_request_id.py | готово |

Вне MVP (этапы 3–4, Could): FR-60–FR-65.
