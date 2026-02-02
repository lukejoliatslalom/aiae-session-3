# Cloud Architecture Overview

This document provides a simple system context view of the monorepo application: a React frontend, an Express API backend, and an in-memory data store (development).

```mermaid
flowchart LR
    %% System Context Diagram (high-level)
    user["User"]:::ext --> browser["React Frontend (Browser)"]:::fe
    browser -->|HTTP/JSON| api["Express API (Node.js)"]:::be
    api --> store[("In-Memory Store\n(e.g., SQLite :memory:)")]:::data

    subgraph Client
      browser
    end

    subgraph Backend
      api --> store
    end

    classDef ext fill:#fff,stroke:#888,color:#212121
    classDef fe fill:#E3F2FD,stroke:#1976d2,color:#0D47A1
    classDef be fill:#FFF3E0,stroke:#ff9800,color:#E65100
    classDef data fill:#F5F5F5,stroke:#9e9e9e,color:#424242
```

Notes
- Environment: Dev container on Ubuntu; backend uses an in-memory store (non-persistent).
- Communication: The React app calls the Express API via HTTP using JSON payloads.
- Persistence: Data resets on server restart; no external DB or cloud services in MVP.
- Next steps: Replace in-memory storage with a persistent database and add auth/gateway if needed.

## Sequence: Create TODO

```mermaid
sequenceDiagram
  participant U as User
  participant FE as React Frontend (Browser)
  participant API as Express API (Node.js)
  participant MEM as In-Memory Store (e.g., SQLite :memory:)

  U->>FE: Enter title, description, due date, priority
  FE->>API: POST /api/tasks {title, description, due_date, priority}
  API->>API: Validate input (title required, normalize date)
  API->>MEM: Insert new task
  MEM-->>API: OK (id, created_at)
  API-->>FE: 201 Created {id, title, description, due_date, priority, completed}
  FE->>FE: Update task list (sorted, filtered)
  FE-->>U: Show success confirmation
```
