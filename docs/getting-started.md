---
sidebar_position: 2
---

# Getting Started

This guide will help you install and configure xatrix on your local machine.

## Prerequisites

Before you begin, ensure you have the following installed:

- **Python 3.13+** - [Download Python](https://www.python.org/downloads/)
- **Ollama** - [Install Ollama](https://ollama.com)
- **Git** - For cloning the repository

## Installation Steps

### 1. Install Ollama

Ollama is required to run the local LLM that powers xatrix.

```bash
# On macOS/Linux
curl -fsSL https://ollama.com/install.sh | sh

# On Windows
# Download from https://ollama.com/download
```

### 2. Pull an AI Model

xatrix works with any Ollama model that supports tool calling:

```bash
# Recommended: Qwen3-30B-A3B (18GB, best tool calling)
ollama pull qwen3:30b-a3b

# Alternative: gpt-oss:120b (70GB, high quality)
ollama pull gpt-oss:120b

# Other options:
ollama pull llama3:70b
ollama pull mistral:latest
```

:::info Model Requirements
- **Tool calling support** is required for xatrix to work
- **Qwen3-30B**: ~18GB, 32GB+ RAM recommended
- **gpt-oss:120b**: ~70GB, 128GB+ RAM recommended
:::

### 3. Clone and Install xatrix

```bash
# Clone the repository
git clone https://github.com/yourusername/xatrix.git
cd xatrix

# Install with pip
pip install -e .

# Or use uv for faster installation
uv pip install -e .
```

### 4. Verify Installation

Check if everything is set up correctly:

```bash
xatrix --check
```

Expected output:
```
✅ Ollama is running
✅ Model qwen3:30b-a3b available
✅ Workspace configured
✅ Memory database initialized
```

## First Run

Start the interactive CLI:

```bash
xatrix
```

You should see the xatrix welcome message and prompt:

```
🤖 xatrix v0.1.0
Type /help for commands or just start chatting!

You:
```

### Try Your First Command

```
You: Hello! What can you do?
Agent: I'm xatrix, your local AI assistant. I can help you with...
```

## Configuration

### Environment Variables

Create a `.env` file in the project root:

```bash title=".env"
# LLM Configuration
OLLAMA_MODEL=gpt-oss:120b
# Or: qwen3:30b-a3b, llama3:70b, mistral:latest
OLLAMA_BASE_URL=http://localhost:11434

# Agent Configuration
AGENT_NAME=xatrix
LOG_LEVEL=info

# WhatsApp Integration (optional)
WHATSAPP_ENABLED=false
WHATSAPP_WEBHOOK_PORT=8743
```

### Workspace Setup

The workspace contains agent-modifiable files:

```
workspace/
├── SOUL.md              # Agent personality and identity
└── skills/              # Skill definitions
    ├── self_improve/
    ├── code_assistant/
    └── memory_manager/
```

:::tip Customize Your Agent
Edit `workspace/SOUL.md` to define your agent's personality, goals, and behavior patterns.
:::

## CLI Commands

| Command    | Description                      |
|------------|----------------------------------|
| `/new`     | Start a new conversation        |
| `/status`  | Show system status              |
| `/skills`  | List available skills           |
| `/help`    | Display help message            |
| `/quit`    | Exit xatrix                    |

## Daemon Mode

Run xatrix as a background daemon:

```bash
# Start daemon
xatrix start

# Check status
xatrix status

# Stop daemon
xatrix stop

# Restart daemon
xatrix restart
```

## Next Steps

- 📚 [Learn about Architecture](/architecture/overview)
- 🛠️ [Create Your First Skill](/skills/creating-skills)
- 🔌 [Set Up WhatsApp Integration](/integrations/whatsapp)
- 📊 [Understand the Job Queue](/architecture/job-queue)
