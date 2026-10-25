# demo

Демо-данные: `python -m app.demo load|reset|clear [--set main|overdue] [--if-empty]` (`make demo`, `make demo-reset`).
Наборы лежат в `backend/demo/<имя>.yaml`. Команды работают только при `DEV_MODE` (см. «Быстрый старт» корневого [README](../../../README.md)). `--if-empty` дополнительно требует `DEMO_DATA=1` и пустую БД.

`DEMO_TEAM_EMAILS` — адреса команды через запятую (пробелы вокруг игнорируются). Демо-участники получают их
по ключу (`ivan`, `anna`, `oleg`, `kate`, порядок `TEAM_SLOTS` в `loader.py`), а не по порядку в файле, поэтому
`overdue` переиспользует тех же людей. Адресов меньше, чем участников, или переменная пуста: остаётся `example.com`.
Вход ведущего `host@example.com / demo1234` не меняется.
