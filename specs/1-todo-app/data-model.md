# Data Model for シンプルなTodoアプリ

This project stores and manipulates a single entity:

## TodoItem

Represents a user-created task.

| Field     | Type    | Description                               | Validation          |
| --------- | ------- | ----------------------------------------- | ------------------- |
| id        | string  | Unique identifier (UUID)                  | Required, unique    |
| title     | string  | Short text describing the task            | Required, non-empty |
| completed | boolean | Whether the task has been marked complete | Defaults to `false` |
| createdAt | string  | ISO timestamp when the task was created   | Auto-populated      |

### Storage Format

Todos are stored as a JSON-serialized array under the `localStorage` key `todos`.

```json
// Example value in localStorage
[
  {
    "id": "c0a80123-4567-89ab-cdef-0123456789ab",
    "title": "Buy milk",
    "completed": false,
    "createdAt": "2026-02-17T12:34:56.789Z"
  }
]
```

### State Transitions

- **Create**: new item added with `completed=false` and current `createdAt`.
- **Complete**: toggling `completed` from `false` to `true`; UI may allow toggling back.
- **Delete**: item removed from array.

### Validation Rules

- `title` must be trimmed and non-empty; maximum length could be capped (e.g. 255 characters).
- `id` generated via stable UUID library.

No other entities are required for this simple app.
