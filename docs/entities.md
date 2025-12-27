# Сущности

## `Task`

### Назначение
Хранит “дело/единицу работы”, которую бот может предлагать пользователю, а UI — отображать и редактировать. Статус отражает **жизненный цикл дела**, а не состояние диалога.

### Поля
- `id` — UUID
- `created_at` — timestamp
- `updated_at` — timestamp
- `parent_id` — nullable FK → `Task.id`
- `title` — string
- `status` — enum:
  - `todo`
  - `in_progress`
  - `blocked`
  - `done`
- `actor` — enum:
  - `bot`
  - `user`
  - `system`
- `source` — enum:
  - `user`
  - `ai`
  - `system`
  - `template`
  - `import`

### Инварианты
- `updated_at` не раньше `created_at`.
- Если `parent_id` задан:
  - `parent_id` должен существовать.
  - `parent_id` не должен быть равен `id`.
- `title` не пустой.
- `status`, `actor` и `source` валидны.

## `TaskDialogEvent`

### Назначение
Неизменяемый журнал событий взаимодействия “бот ⇄ пользователь” вокруг задач: предложения, согласия, старты, отказы, завершения. Источник истины для поведения бота и аналитики.

### Поля
- `id` — UUID
- `created_at` — timestamp
- `task_id` — FK → `Task.id`
- `type` — enum:
  - `task_proposed`
  - `task_accepted`
  - `task_started`
  - `task_progress`
  - `task_refused`
  - `task_done`
  - `task_status_set` _(опционально)_
- `actor` — enum:
  - `bot`
  - `user`
  - `system`

### Инварианты
- События append-only (не редактируются и не удаляются).
- `task_id` должен существовать.
- `type` и `actor` валидны.

## `UserStateEvent`

### Назначение
Неизменяемый журнал смены состояния пользователя, влияющего на поведение бота: сон, работа/быт, отдых, недоступность (игнорирование). Не привязан к конкретной задаче.

### Поля
- `id` — UUID
- `created_at` — timestamp
- `type` — enum:
  - `state_set`
  - `state_cleared`
- `state` — nullable enum:
  - `work`
  - `household`
  - `rest`
  - `sleep`
  - `unavailable`
- `actor` — enum:
  - `bot`
  - `user`
  - `system`

### Инварианты
- События append-only (не редактируются и не удаляются).
- `type`, `state` (если не null) и `actor` валидны.
- Если `type` требует `state` (например, `state_set`), `state` не null.
