# Product Requirements Document (PRD) - TODO App Upgrade: Due Dates, Priorities, Filters

## 1. Overview

We are upgrading the basic TODO app to support due dates, priorities, and simple filters so users can better organize tasks and quickly identify what needs attention. The MVP keeps implementation lean with local-only storage and no backend changes, aligning with stakeholder guidance from the requirements meeting and Slack follow-up.

---

## 2. MVP Scope

- Data model & validation:
  - `title`: required.
  - `priority`: "P1" | "P2" | "P3"; default "P3".
  - `dueDate`: optional ISO `YYYY-MM-DD`; invalid values ignored (treated as absent).
- Due dates: Add optional `dueDate` to tasks.
- Priority: Support three levels `P1`, `P2`, `P3` with default `P3`.
- Filters: Provide views for **All**, **Today**, and **Overdue**.
- Storage: Local-only; no backend or external storage; no backend changes.

---

## 3. Post-MVP Scope

- Overdue highlighting: Visually emphasize overdue tasks (e.g., red highlight).
- Priority badges: Color-coded indicators (e.g., red `P1`, orange `P2`, gray `P3`).
- Sorting rules: Overdue first → priority (P1→P3) → due date ascending → undated last.
- Filter behavior: In **Today** and **Overdue**, show only incomplete tasks; completed tasks remain visible in **All**.

---

## 4. Out of Scope

- Notifications.
- Recurring tasks.
- Multi-user features.
- Keyboard navigation and advanced accessibility beyond basic UI.
- External storage or backend changes (remain local-only).
