# llm

Промпты с версиями (`llm_prompts`), метрики вызовов (`llm_calls`) и разбор строки нейросетью (FR-12, FR-21).

| Ручка | FR |
| --- | --- |
| `GET /admin/llm/prompts` | контракт LLM |
| `POST /admin/llm/prompts` (новая версия, одна активная на задачу) | FR-12, OQ-07 |

Сервисные функции `parse_text` и `enrich_item` готовы, но обогащение через LLM к `POST /meetings/{id}/items` не подключено: правила отвечают сразу, LLM-часть станет фоновой задачей Celery.

Клиент (`client.py`) — протокол `LLMClient`, реализация `OpenAICompatClient` и заглушка `StubLLMClient`; заглушка включается при пустых `LLM_API_KEY` или `LLM_MODEL`. Дедлайн вызова и порядок повторов — в [docs/decisions.md](../../../../docs/decisions.md).

Ответ нейросети проверяется: поле, не прошедшее проверку, отбрасывается, `manual_fields` не меняются. Текст в `llm_calls` и в логи не пишется. Стартовый промпт `parse` v1 приходит миграцией `0005`.

`/admin/*` защищён `X-Admin-Token`, подробности — в [analytics](../analytics/README.md).
