# 1. Индекс standing-rules в omo

> **Происхождение:** снимок mcode (`/home/penis/projects/mcode-extensions/docs/standing-rules/`), адаптированный под omo.
> Разделы `02-desired-behavior.md` и `03-contracts.md` перенесены как содержание контракта; `01-inventory.md`, `04-bindings.md`, `coverage.md` и `capabilities.json` **не дублируются** в omo — они остаются снимок mcode по пути репозитория и не описывают omo-каталог.

## Что здесь лежит

| Файл | Что это | Владелец |
| --- | --- | --- |
| `02-desired-behavior.md` | Желаемое поведение агента: каким наблюдаемым результатом считается работа | этот каталог |
| `03-contracts.md` | Исполнимые контракты: общие шаги S1–S6, таблица правил, таблица доставки | этот каталог |
| `../existing-ports-contract.md` | Контракт существующих портов omo (control-task, agent-state, injection-rules, system-status, probe-hook) | `~/.omo/agent/docs/` |

Каталог `~/.omo/agent/docs/standing-rules/` — **единственный** в omo; дубликата в mcode-репозитории нет и не требуется.

## Перенесённый набор правил

Правила живут не здесь, а в каталоге-владельце `~/.omo/agent/rules/` — по одному файлу `*.json` на правило. Каждый файл проходит валидатор `parseRuleFile` (`~/.omo/agent/extensions/lib/injection-rules.ts`); этот валидатор — единственный авторитет формата.

| Правило | Постоянная доставка | Напоминание |
| --- | --- | --- |
| 01-operating-principle | доставка на старте main/child и после compaction | 1 шаг |
| 17-user-correction-rebase | доставка на старте main/child и после compaction | на каждом вводе пользователя |
| 68-reason-in-user-language | доставка на старте main/child и после compaction | на каждом вводе пользователя + 37 шаг |
| 70-reasoning-poison | доставка на старте main/child и после compaction | на каждом вводе пользователя + 2 шаг (полное тело) |
| 71-user-intent-outcome | доставка на старте main/child и после compaction | 2 шаг |
| 72-project-sources-details | доставка на старте main/child и после compaction | 5 шаг |
| 73-tests-not-proof | доставка на старте main/child и после compaction | 3 шаг |
| 74-use-context-frontier | доставка на старте main/child и после compaction | 7 шаг |
| 75-evidence-or-blocker | доставка на старте main/child и после compaction | 8 шаг |
| 76-execution-progress-close-questions | доставка на старте main/child и после compaction | 5 шаг |
| 77-layout-model-and-verification | доставка на старте main/child и после compaction | 10 шаг |
| 79-behavior-comments-and-contract | доставка на старте main/child и после compaction | 12 шаг |
| 80-independent-review | доставка на старте main/child и после compaction | 13 шаг |
| 81-engineering-invariants | доставка на старте main/child и после compaction | 17 шаг |

## Модель доставки в omo

Полный контракт хранится в `body` и доставляется main/child при старте, возобновлении и после принятого compaction. `remind.suffix` — отдельное короткое напоминание; `remind.send_full_rule: true` выбирает вместо него полный `body`. Отсутствующие `send_on_start`/`send_on_subagent` означают `true`, явное `false` выключает соответствующую доставку. `apply` проверяется для реального имени роли; child с `send_on_subagent: false` не получает и reminders.

Принятая OMO каденция `every_n_turns` считает завершённые шаги модели, включая шаги с инструментами, независимо от числа запросов пользователя. Счётчик не использует сбрасываемый Native `turnIndex`; при открытии сессии восстанавливается из полного пути истории, включая скрытые compaction сообщения. `every_input` — независимый путь на каждом пользовательском вводе, включая slash-команды и queued/streaming ввод. Внутренний `source: extension` не считается пользовательским вводом. Rule68 совмещает ввод и37шагов, rule70 — ввод и2шага; rule70 всегда доставляет полное тело. Legacy `on_user_end` остаётся явной input-clock настройкой сторонних правил и не требуется для `every_input`.

Исходный пользовательский ввод никогда не изменяется для доставки правил: обычные сообщения, slash-команды, queued/streaming ввод и сохранённая история остаются пользовательским текстом. Все reminders поступают отдельно через существующий context-inject с `channel: instruction`: OpenAI — отдельный system/developer turn; Anthropic — top-level `system` text block; Google Generative AI/Vertex — `config.systemInstruction`. Этот строгий канал не использует fallback `role: user`. После принятого compaction он восстанавливает полные тела, даже если run продолжается без нового `before_agent_start`. Неизвестный API/shape явно сообщает незавершённую доставку; pending reminders сохраняются для повторной доставки. Native runtime сообщает ошибки extension handlers, но сам не гарантирует остановку provider request при такой ошибке. Обычный `channel: system` других context sources сохраняет свою прежнюю совместимость. Доставка текста не доказывает послушание модели.

## Чего здесь нет и почему

- **Машинный инвентарь** (`01-inventory.md`, `04-bindings.md`, `coverage.md`, `capabilities.json`) описывает каталог mcode (skills, tools, MCP на снимок 2026-10-22). Пересборка этого инвентаря под omo-каталог — отдельная исследовательская задача; в omo он намеренно не дублируется, чтобы не возникло второй, дрейфующей копии.
- **Привязки к конкретным tools/skills** живут в самих файлах правил (`body` содержит локальные процедуры и подходящие инструменты) и в `docs/standing-rules/04-bindings.md` репозитория mcode как справочник discovery. Каталог помогает найти возможность; актуальная schema решает вызов.
- **Расписание фаз 3** (роутинг пайплайна) — не входит в этот каталог: локальный `task_pipeline` удалён владельцем 2026-10-28, задачным циклом владеет общий скилл `~/.agents/skills/task`.
