# Epics and Stories - TODO App Upgrade

Note: Content is based on requirements in docs/prd-todo.md and follows the structure defined in docs/templates/epic-and-stories-template.md.

## MVP

- Epic: Core Data Model Update
  - Story: Add priority field with default P3
  - Story: Add optional dueDate (YYYY-MM-DD)
  - Story: Ignore invalid dueDate values
  - Story: Title field is required
  - Technical Requirements:
    - Backend (`packages/backend/src/app.js`): Add `priority TEXT CHECK(priority IN ('P1','P2','P3')) DEFAULT 'P3'` to `tasks`; update POST/PUT to validate `title`, accept `description`, `due_date` (ISO `YYYY-MM-DD` only) and `priority`; normalize invalid `due_date` to `NULL`.
    - Backend: Keep existing endpoints (`GET/POST/PUT/PATCH/DELETE /api/tasks`) and JSON keys (`due_date`, `completed`).
    - Frontend (`packages/frontend/src/TaskForm.js`): Add priority selector (P1/P2/P3); continue sending `{ title, description, due_date }` plus `priority` to API; enforce title required and normalize date to `YYYY-MM-DD`.
    - Frontend (`packages/frontend/src/TaskList.js`): Continue rendering description and due date chip; prepare for priority display.

- Epic: Task Filters
  - Story: Implement All view
  - Story: Implement Today view
  - Story: Implement Overdue view
  - Technical Requirements:
    - Frontend: Add tabbed navigation (All/Today/Overdue) and client-side filtering; default to All.
    - Frontend: Compute Today/Overdue using `due_date` vs local date and `completed === 0`.
    - Backend: Optionally support query `completed=true|false` and `search` (already implemented) for All view; Today/Overdue can be client-side without backend changes.

- Epic: Priority Levels
  - Story: Support P1, P2, P3 levels
  - Technical Requirements:
    - Data: Enum values `P1 | P2 | P3` with default `P3`.
    - Frontend: Add MUI select control in `TaskForm` and include `priority` in POST/PUT bodies.
    - Backend: Validate `priority` and persist in `tasks` table.

- Epic: Local-Only Storage
  - Story: Persist tasks using local storage
  - Story: No backend changes or external storage
  - Technical Requirements:
    - Given current implementation uses an in-memory SQLite backend, continue using `/api/tasks` without external persistence; avoid adding remote storage or multi-user features.
    - If local-only persistence is required later, mirror API behavior via `localStorage` while keeping UI contract stable.

## Post-MVP

- Epic: Overdue Highlighting
  - Story: Visually highlight overdue tasks
  - Technical Requirements:
    - Frontend (`TaskList`): Mark tasks as overdue when `due_date < today` and `completed === 0`; apply red-toned styles per UI guidelines.

- Epic: Priority Badges
  - Story: Add color-coded badge for P1 (red)
  - Story: Add color-coded badge for P2 (orange)
  - Story: Add color-coded badge for P3 (gray)
  - Technical Requirements:
    - Frontend: Render MUI `Chip` badges with colors mapped to priority; read from `task.priority`.

- Epic: Task Sorting Rules
  - Story: Sort overdue tasks first
  - Story: Sort by priority P1→P3
  - Story: Sort by due date ascending
  - Story: Place undated tasks last
  - Technical Requirements:
    - Backend: Update `GET /api/tasks` ORDER BY to: overdue first (CASE on `completed` and `due_date < date('now')`), then priority (map `P1<P2<P3`), then `due_date` ASC, then `created_at` ASC.
    - Alternative: Perform the same sort client-side in `TaskList` if backend remains generic.

- Epic: Filter Behavior for Completion
  - Story: Show only incomplete tasks in Today
  - Story: Show only incomplete tasks in Overdue
  - Story: Show completed tasks in All
  - Technical Requirements:
    - Frontend: Exclude `completed === 1` from Today/Overdue views; include in All.
    - Backend: Existing `completed=true|false` query parameter can support All view filtering when needed.
