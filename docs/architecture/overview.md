---
sidebar_position: 1
---

# Architecture Overview

xatrix follows a modular architecture with clear separation of concerns. The system is built around four core components that work together to create a self-improving AI agent.

## System Components

```mermaid
graph TB
    Interface[Interface Layer<br/>CLI / API / Channels]
    Harness[Harness<br/>Orchestrator & Session Manager]
    Brain[Brain<br/>LLM via Ollama]
    Tools[Tools & Skills<br/>Filesystem, Bash, Memory]
    Workspace[Workspace<br/>SOUL.md + Skills]
    
    Interface --> Harness
    Harness --> Brain
    Brain --> Tools
    Harness --> Workspace
    Tools --> Workspace
```

## Core Components

### 1. Interface Layer

Multiple ways to interact with the agent:

- **CLI** (`xatrix.interface.cli`) - Terminal-based chat interface
- **Daemon** (`xatrix.daemon`) - Background service mode
- **Channels** (`xatrix.channels`) - WhatsApp, Telegram, Discord integrations
- **API** - FastAPI REST endpoints (planned)

### 2. Harness (Orchestrator)

**File**: `xatrix/harness.py`

The harness is the heart of xatrix. It:

- **Loads System Prompt** from `xatrix/SOUL.md` + active skills
- **Manages Conversation Flow** with session persistence
- **Executes Tool Calls** from the LLM with validation
- **Coordinates Job Queue** for async task processing
- **Protects Core System** - agent branch isolation via git

:::tip Immutable Core
The harness itself cannot be modified by the agent, ensuring system stability.
:::

### 3. Brain (LLM Interface)

**File**: `xatrix/brain.py`

Abstracts LLM communication:

```python
from xatrix.brain import Brain

brain = Brain()
response = brain.complete(
    messages=[{"role": "user", "content": "Hello"}],
    tools=available_tools,
    stream=True
)
```

**Features:**
- Streaming responses
- Tool-calling support (native function calling)
- Token counting and context management
- Configurable temperature, top-p, etc.

**Current Model**: Qwen3-30B-A3B via Ollama

### 4. Tools & Skills System

**Primitive Tools** (`xatrix/tools/`):
- `filesystem.py` - read, write, edit, list, grep
- `bash.py` - Safe shell command execution (allowlist)
- `memory_tool.py` - Memory storage and retrieval
- `registry.py` - Tool registration and validation

**Skills** (`xatrix/skills/`):
- Defined in `SKILL.md` files
- Loaded dynamically into system prompt
- Agent can create new skills
- No code changes needed

## Data Flow

### User Request Flow

```mermaid
sequenceDiagram
    participant U as User
    participant I as Interface
    participant H as Harness
    participant B as Brain (LLM)
    participant T as Tools

    U->>I: Input
    I->>H: Forward message
    H->>H: Build context (SOUL + skills + history)
    H->>B: messages + tool schemas
    alt Tool call
        B->>H: tool_call request
        H->>T: execute(tool, args)
        T-->>H: result
        H->>B: tool result
        B->>H: final text response
    else Direct response
        B->>H: text response
    end
    H->>I: response
    I->>U: Reply
```

### Job Queue Flow

For async/long-running tasks:

```mermaid
graph LR
    A[User Request] -->|enqueue| B[(Job Queue)]
    B -->|dequeue| C[Worker Thread]
    C -->|process| D[Harness]
    D -->|result| E[Notify / Reply]
```

See [Job Queue System](/architecture/job-queue) for details.

## Memory Architecture

**File**: `xatrix/memory/store.py`

SQLite-based persistence:

```
data/
└── xatrix.db
    ├── conversations      # Chat history per session
    ├── facts              # User preferences, learned info
    ├── state              # Key-value agent state
    ├── scheduled_tasks    # Cron-like scheduled tasks
    └── events             # Audit log of all tool calls
```

Memory tools allow the agent to:
- Remember user preferences
- Track conversation context
- Store learned information
- Build knowledge over time

## Security Model

### Protected vs. Modifiable

**Protected (base branch):**
- `xatrix/` - Core Python package
- `bridges/` - Integration services

**Agent branch (`xatrix/auto`):**
- `xatrix/skills/` - Skills and SOUL.md
- `xatrix/tools/` - Dynamic tools
- `data/` - Database, logs

### Tool Safety

- Bash commands use **allowlist** (grep, ls, cat, etc.)
- Filesystem operations are **sandboxed** to workspace
- No network access tools by default
- All tool calls are **logged**

## Configuration

**File**: `xatrix/config.py`

Centralized configuration with environment variable support:

```python
class Config:
    MODEL_NAME: str = "qwen3:30b-a3b"
    OLLAMA_URL: str = "http://localhost:11434"
    WORKSPACE_DIR: Path = Path("workspace")
    DATA_DIR: Path = Path("data")
    MAX_HISTORY_LENGTH: int = 20
```

## Next Steps

- [Creating Skills](/skills/creating-skills) - Learn how to create custom skills
- [WhatsApp Integration](/integrations/whatsapp) - Set up WhatsApp channel
- [Job Queue System](/architecture/job-queue) - Understand async task processing
