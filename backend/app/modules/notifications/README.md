# notifications

Письма: SUMMARY, CONFIRM_REQUEST, CONFIRM_REPEAT, HOST_RESPONSE, DUE_TOMORROW, ITEM_CHANGED, ITEM_REASSIGNED, PASSWORD_RESET. Ручек нет: письма ставят в очередь сервисы встречи и пунктов, отправляет Celery Beat. Без `SMTP_HOST` письма только пишутся в лог.

Очередь: `service.queue` не создаёт повторов по `dedupe_key` (форматы — в [BR.md](../../../../docs/requirements/BR.md)), `queue_item_changed` сливает правки до отправки в одно письмо (FR-16, BR-16), `plan_due_reminder`, `replan_due_tomorrow` и `plan_reassign_repeat` планируют напоминания (BR-8, BR-10).

Отправка: Beat раз в минуту вызывает `sender.send_due`, она забирает письма пачкой и передаёт `sender.deliver`. Лимит писем, повторы при сбое и защита от двойной отправки описаны в [docs/decisions.md](../../../../docs/decisions.md).

`content.py` собирает данные письма, `render.py` — тему, текст и HTML. Ссылку выдаёт `confirm.service.ensure_link`, а для каждого письма отправитель просит у `confirm.service.issue_token` новый токен; в базе остаётся только хеш.
