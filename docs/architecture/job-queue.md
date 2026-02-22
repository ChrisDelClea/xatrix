---
sidebar_position: 4
---

# Job Queue

The job queue enables asynchronous, prioritized task processing. All incoming requests — from WhatsApp, the scheduler, or the CLI — are enqueued as jobs and processed sequentially by a worker thread.

## Architecture

```mermaid
graph TD
    WA[WhatsApp Channel] -->|enqueue| Q[(Job Queue\nSQLite)]
    SC[Scheduler] -->|enqueue| Q
    CLI[CLI / API] -->|enqueue| Q
    Q -->|dequeue by priority| W[Worker Thread\nsequential]
    W -->|process| H[Harness.process]
    H -->|response| OUT[Send Reply]
```

## Job Flow

```mermaid
sequenceDiagram
    participant Source as Event Source
    participant Queue as Job Queue
    participant Worker as Worker Thread
    participant Harness as Harness

    Source->>Queue: enqueue(job)
    Note over Queue: status = pending
    Worker->>Queue: dequeue (highest priority)
    Note over Queue: status = running
    Worker->>Harness: process(user_input, context)
    Harness-->>Worker: response
    Note over Queue: status = completed
    Worker->>Source: send reply
```

## Priority Levels

| Priority | Value | Used for |
|----------|-------|----------|
| `urgent` | 10 | System alerts, critical tasks |
| `high` | 7 | WhatsApp voice messages |
| `normal` | 5 | Regular WhatsApp messages |
| `low` | 3 | Scheduled background tasks |

## Jobs Table

```sql
CREATE TABLE jobs (
    id           INTEGER PRIMARY KEY,
    source       TEXT NOT NULL,     -- 'whatsapp', 'scheduler', 'cli'
    priority     INTEGER DEFAULT 5,
    status       TEXT DEFAULT 'pending',  -- pending/running/completed/failed
    payload      TEXT NOT NULL,     -- JSON: {user_input, context}
    created_at   REAL NOT NULL,
    started_at   REAL,
    completed_at REAL,
    retries      INTEGER DEFAULT 0
);
```

## Retry Behavior

Failed jobs are automatically retried up to 3 times with exponential backoff:

```mermaid
stateDiagram-v2
    [*] --> pending
    pending --> running: Worker picks up
    running --> completed: Success
    running --> failed: Error (retries = 3)
    running --> pending: Error (retries < 3)
    failed --> [*]
    completed --> [*]
```

