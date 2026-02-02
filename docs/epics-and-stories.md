# Epics, Stories, Acceptance Criteria, and Technical Requirements - TODO App Upgrade

Note: Content is based on requirements in docs/prd-todo.md and follows the structure defined in docs/templates/epic-and-stories-template.md.

## Epics & Stories

## MVP

- Epic: Core Data Model Update
  - Story: Title field is required
    - Acceptance Criteria:
      - Prevent submit and show inline error when `title` is empty.
      - Backend returns 400 with `{ error: 'Task title is required' }`.
      - Error messaging is accessible and announced to screen readers.
  - Story: Add optional dueDate (YYYY-MM-DD)
    - Acceptance Criteria:
      - `dueDate` is optional; normalized to ISO `YYYY-MM-DD`.
      - Invalid or empty values are stored as `null` and do not block creation.
      - Due date displays in list as a formatted chip when present.
  - Story: Ignore invalid dueDate values
    - Acceptance Criteria:
      - Invalid dates (non-ISO, impossible values) are rejected client-side.
      - Backend normalizes invalid `due_date` to `NULL` without erroring.
      - UI shows no due date chip when value is invalid or `null`.
  - Story: Add priority field with default P3
    - Acceptance Criteria:
      - `priority` supports `P1 | P2 | P3` with default `P3`.
      - Priority is editable in the form and persisted via POST/PUT.
      - Invalid values are rejected; form shows validation message.
      - Priority is available for display in lists (badge/chip).
  - Technical Requirements:
    - Backend (`packages/backend/src/app.js`): Add `priority TEXT CHECK(priority IN ('P1','P2','P3')) DEFAULT 'P3'` to `tasks`; update POST/PUT to validate `title`, accept `description`, `due_date` (ISO `YYYY-MM-DD` only) and `priority`; normalize invalid `due_date` to `NULL`.
    - Backend: Keep existing endpoints (`GET/POST/PUT/PATCH/DELETE /api/tasks`) and JSON keys (`due_date`, `completed`).
    - Frontend (`packages/frontend/src/TaskForm.js`): Add priority selector (P1/P2/P3); continue sending `{ title, description, due_date }` plus `priority` to API; enforce title required and normalize date to `YYYY-MM-DD`.
    - Frontend (`packages/frontend/src/TaskList.js`): Continue rendering description and due date chip; prepare for priority display.

- Epic: Task Filters
  - Story: Implement All view
    - Acceptance Criteria:
      - Default tab shows all tasks regardless of completion state.
      - Completed tasks are visible and identifiable.
  - Story: Implement Today view
    - Acceptance Criteria:
      - Shows tasks due today and not completed (`completed === 0`).
      - Date comparisons are timezone-aware and consistent.
  - Story: Implement Overdue view
    - Acceptance Criteria:
      - Shows tasks with `due_date` before today and not completed.
      - Overdue state is consistent with sorting/highlighting rules.
  - Story: Keyword search (title/description)
    - Acceptance Criteria:
      - Filters tasks by keyword across title and description.
      - Case-insensitive; updates results with minimal latency.
  - Technical Requirements:
    - Frontend: Add tabbed navigation (All/Today/Overdue) and client-side filtering; default to All.
    - Frontend: Compute Today/Overdue using `due_date` vs local date and `completed === 0`.
    - Backend: Optionally support query `completed=true|false` and `search` (already implemented) for All view; Today/Overdue can be client-side without backend changes.

- Epic: Priority Levels
  - Story: Support P1, P2, P3 levels
    - Acceptance Criteria:
      - Enum values `P1 | P2 | P3` enforced end-to-end.
      - Priority can be set/edited in the form.
      - Priority displays consistently in lists/detail views.
  - Technical Requirements:
    - Data: Enum values `P1 | P2 | P3` with default `P3`.
    - Frontend: Add MUI select control in `TaskForm` and include `priority` in POST/PUT bodies.
    - Backend: Validate `priority` and persist in `tasks` table.

- Epic: Local-Only Storage
  - Story: Persist tasks using current local API contract
    - Acceptance Criteria:
      - Continue using `/api/tasks` without introducing external persistence.
      - No multi-user features; scope remains single-user local development.
  - Story: No backend changes or external storage
    - Acceptance Criteria:
      - Avoid remote storage or third-party services.
      - UI contract remains stable during MVP.
  - Technical Requirements:
    - Given current implementation uses an in-memory SQLite backend, continue using `/api/tasks` without external persistence; avoid adding remote storage or multi-user features.

## Post-MVP

- Epic: Overdue Highlighting
  - Story: Visually highlight overdue tasks
    - Acceptance Criteria:
      - Overdue tasks (not completed, `due_date` < today) are visually emphasized per UI guidelines.
      - Colors and emphasis meet WCAG AA contrast and accessibility.
  - Technical Requirements:
    - Frontend (`TaskList`): Mark tasks as overdue when `due_date < today` and `completed === 0`; apply red-toned styles per UI guidelines.

- Epic: Priority Badges
  - Story: Add color-coded badge for P1 (red)
    - Acceptance Criteria:
      - P1 displays a red badge; consistent with the palette.
  - Story: Add color-coded badge for P2 (orange)
    - Acceptance Criteria:
      - P2 displays an orange badge.
  - Story: Add color-coded badge for P3 (gray)
    - Acceptance Criteria:
      - P3 displays a gray badge.
  - Technical Requirements:
    - Frontend: Render MUI `Chip` badges with colors mapped to priority; read from `task.priority`.

- Epic: Task Sorting Rules
  - Story: Sort overdue tasks first
    - Acceptance Criteria:
      - Overdue incomplete tasks appear before non-overdue tasks.
  - Story: Sort by priority P1→P3
    - Acceptance Criteria:
      - Within the same overdue bucket, tasks sort P1, then P2, then P3.
  - Story: Sort by due date ascending
    - Acceptance Criteria:
      - Within the same priority, earlier due dates appear first.
  - Story: Place undated tasks last
    - Acceptance Criteria:
      - Tasks without a due date are listed after dated tasks.
  - Technical Requirements:
    - Backend: Update `GET /api/tasks` ORDER BY to: overdue first (CASE on `completed` and `due_date < date('now')`), then priority (map `P1<P2<P3`), then `due_date` ASC, then `created_at` ASC.
    - Alternative: Perform the same sort client-side in `TaskList` if backend remains generic.

- Epic: Filter Behavior for Completion
  - Story: Show only incomplete tasks in Today
    - Acceptance Criteria:
      - Completed tasks are excluded from Today.
  - Story: Show only incomplete tasks in Overdue
    - Acceptance Criteria:
      - Completed tasks are excluded from Overdue.
  - Story: Show completed tasks in All
    - Acceptance Criteria:
      - All view includes completed tasks.
  - Technical Requirements:
    - Frontend: Exclude `completed === 1` from Today/Overdue views; include in All.
    - Backend: Existing `completed=true|false` query parameter can support All view filtering when needed.
