┌─────────────────┐
│ Event Sources   │
├─────────────────┤
│ WhatsApp       │──┐
│ Scheduler      │──┼──→ Job Queue (SQLite)
│ CLI            │──┘      ↓
└─────────────────┘    [pending jobs]
                           ↓
                    Worker Thread
                     (sequenziell)
                           ↓
                    Harness.process()


Jobs-Tabelle:

sql
jobs:
  - id
  - source (whatsapp, scheduler, cli)
  - priority (urgent=10, high=7, normal=5, low=3)
  - status (pending, running, completed, failed)
  - payload (JSON: user_input, context)
  - created_at
  - started_at
  - completed_at
  - retries

