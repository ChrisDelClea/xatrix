---
sidebar_position: 2
---

# Memory System

xatrix's memory system provides persistent storage for conversations, facts, and agent state using SQLite. This allows the agent to remember information across sessions and build knowledge over time.

## Architecture

**File**: `xatrix/memory/store.py`

The `MemoryStore` class provides a SQLite-backed storage layer with the following tables:

```
data/agent.db
├── conversations      # Chat history per session
├── facts             # User preferences, learned information
├── state             # Key-value state storage
├── scheduled_tasks   # Cron-like scheduled tasks
└── events            # Audit log of tool calls and actions
```

## Core Components

### 1. Conversations

Stores full message history including tool calls:

```python
from xatrix.memory.store import MemoryStore

store = MemoryStore()

# Save a message
store.save_message(
    session_id="session123",
    role="user",
    content="What's the weather today?"
)

# Retrieve conversation history
messages = store.get_conversation("session123", limit=50)
# Returns: [{"role": "user", "content": "...", "timestamp": 1234567890}, ...]

# Clear conversation
store.clear_conversation("session123")
```

**Schema:**
```sql
CREATE TABLE conversations (
    id          INTEGER PRIMARY KEY,
    session_id  TEXT    NOT NULL,
    role        TEXT    NOT NULL,        -- "user", "assistant", "tool"
    content     TEXT,                     -- Message text
    tool_calls  TEXT,                     -- JSON array of tool calls
    tool_call_id TEXT,                    -- For tool responses
    timestamp   REAL    NOT NULL
);
```

### 2. Facts (Long-term Memory)

Key-value storage for persistent information:

```python
# Store a fact
store.store_fact(
    key="user_preference_language",
    value="TypeScript",
    category="preferences"
)

# Recall a specific fact
fact = store.recall_fact("user_preference_language")
# Returns: {"key": "...", "value": "TypeScript", "category": "preferences"}

# Search facts (LIKE query)
results = store.search_facts("TypeScript", limit=10)

# List all facts (optionally by category)
all_facts = store.list_facts()
prefs = store.list_facts(category="preferences")

# Delete a fact
store.delete_fact("user_preference_language")
```

**Schema:**
```sql
CREATE TABLE facts (
    key         TEXT PRIMARY KEY,
    value       TEXT NOT NULL,
    category    TEXT NOT NULL DEFAULT 'general',
    created_at  REAL NOT NULL,
    updated_at  REAL NOT NULL
);
```

**Common Categories:**
- `preferences` - User preferences and settings
- `user` - Information about the user
- `project` - Project-specific context
- `general` - General knowledge

### 3. State (Key-Value Store)

Simple persistent state for agent configuration:

```python
# Set state
store.set_state("last_session", "abc123")
store.set_state("active_skill", "code_assistant")

# Get state
session = store.get_state("last_session", default="")
```

### 4. Scheduled Tasks

Store and manage cron-like scheduled tasks:

```python
# Save a task
store.save_task(
    name="morning_briefing",
    schedule="0 8 * * *",  # Cron format
    prompt="Give me a morning briefing",
    next_run=1234567890.0
)

# Get due tasks
due_tasks = store.get_due_tasks()
# Returns tasks where next_run <= now

# Mark task complete
store.mark_task_done("morning_briefing", next_run=1234567890.0)

# Disable one-time task
store.disable_task("morning_briefing")

# List all tasks
tasks = store.list_tasks()
```

### 5. Events (Audit Log)

Structured logging of agent actions:

```python
# Log an event
store.log_event(
    event_type="tool_call",
    data={
        "tool": "bash",
        "command": "ls -la",
        "result": "success"
    },
    session_id="session123"
)

# Query events
tool_events = store.query_events(
    event_type="tool_call",
    limit=50
)

# All events for a session
session_events = store.query_events(
    session_id="session123"
)
```

**Event Types:**
- `tool_call` - Tool execution
- `file_write` - File creation/modification
- `file_edit` - File editing
- `skill_load` - Skill loading
- `error` - Error occurrences

## Memory Tools

The agent interacts with memory through tools defined in `xatrix/tools/memory_tool.py`:

### Available Tools

#### `memory_store`
Store a fact persistently:
```python
memory_store(
    key="user_timezone",
    value="Europe/Berlin",
    category="user"
)
```

#### `memory_recall`
Retrieve a stored fact:
```python
memory_recall(key="user_timezone")
# Returns: "[user] user_timezone: Europe/Berlin"
```

#### `memory_search`
Search all facts:
```python
memory_search(query="Berlin", limit=10)
```

#### `memory_list`
List all facts:
```python
memory_list()                    # All facts
memory_list(category="user")     # Filtered by category
```

#### `memory_delete`
Delete a fact:
```python
memory_delete(key="user_timezone")
```

## Usage in Skills

Skills can leverage memory tools to create stateful behavior:

```markdown title="workspace/skills/preference_tracker/SKILL.md"
# Preference Tracker

Remember and apply user preferences.

## When to use
- User states a preference
- User asks "Do you remember...?"

## How to proceed
1. Detect preference statement
2. Store using memory_store:
   - Key: "preference:topic"
   - Value: The preference
   - Category: "preferences"
3. Recall when relevant using memory_search

## Examples

User: "I prefer TypeScript over JavaScript"
Agent:
1. Call: memory_store("preference_language", "TypeScript", "preferences")
2. Respond: "Got it! I'll use TypeScript in examples."

User: "Can you help with an API?"
Agent:
1. Call: memory_search("preference_language")
2. Use TypeScript in code examples
```

## How Harness Uses Memory

The harness (`xatrix/harness.py`) integrates memory automatically:

```python
# Load conversation history
messages = self.memory.get_conversation(session_id, limit=20)

# Save user message
self.memory.save_message(session_id, "user", user_input)

# Save assistant response
self.memory.save_message(session_id, "assistant", response)

# Save tool calls
self.memory.save_message(
    session_id,
    "assistant",
    content=None,
    tool_calls=[{"id": "...", "type": "function", ...}]
)

# Save tool results
self.memory.save_message(
    session_id,
    "tool",
    content=result,
    tool_call_id="call_abc123"
)
```

## Database Location

Default: `data/agent.db`

Configured in `xatrix/config.py`:

```python
class MemoryConfig:
    db_path: Path = Path("data") / "agent.db"
```

Override with environment variable:
```bash
MEMORY_DB_PATH=/custom/path/agent.db
```

## Backup and Migration

### Backup
```bash
# SQLite database is a single file
cp data/agent.db data/agent.db.backup
```

### Query directly
```bash
sqlite3 data/agent.db

# List tables
.tables

# View facts
SELECT * FROM facts;

# Search conversations
SELECT * FROM conversations WHERE session_id = 'session123';

# Query events
SELECT * FROM events WHERE type = 'tool_call' ORDER BY timestamp DESC LIMIT 10;
```

## Performance Considerations

### Indexing
All frequently queried columns are indexed:
- `conversations.session_id`
- `conversations.timestamp`
- `facts.category`
- `events.type`, `events.session_id`, `events.timestamp`

### WAL Mode
Write-Ahead Logging enabled for better concurrency:
```python
PRAGMA journal_mode=WAL
```

### Connection Pooling
Single connection with `check_same_thread=False` for daemon mode.

## Best Practices

### Fact Categories
Organize facts into meaningful categories:
```python
# ✅ Good
memory_store("user_name", "Chris", "user")
memory_store("project_language", "Python", "project")
memory_store("preference_editor", "VSCode", "preferences")

# ❌ Bad
memory_store("misc_data", "...", "general")
```

### Key Naming
Use descriptive, consistent key names:
```python
# ✅ Good
"user_timezone"
"preference_language"
"project_deadline"

# ❌ Bad
"tz"
"lang"
"dl"
```

### Memory Lifecycle
- **Conversations**: Auto-managed by harness
- **Facts**: Agent-managed via memory tools
- **State**: System-level configuration
- **Events**: Auto-logged for audit trail

## Extending Memory

Add custom tables by extending `MemoryStore`:

```python
class ExtendedMemoryStore(MemoryStore):
    def _init_schema(self):
        super()._init_schema()
        self._conn.executescript("""
            CREATE TABLE IF NOT EXISTS custom_data (
                id INTEGER PRIMARY KEY,
                data TEXT NOT NULL
            );
        """)
        self._conn.commit()
```

## Next Steps

- [Tools System](/architecture/tools) - How tools integrate with memory
- [Creating Skills](/skills/creating-skills) - Build stateful skills with memory
- [Job Queue System](/architecture/job-queue) - Async task processing
