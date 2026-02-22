---
sidebar_position: 1
slug: /
---

# Welcome to xatrix

**xatrix** is a self-modifying AI agent framework built in Python. It runs **completely locally** using Ollama, with no cloud dependencies or API keys required.

:::tip What makes xatrix unique?

- 🧠 **Local-first**: All processing happens on your machine via Ollama
- 🛠️ **Skills over Tools**: Minimal tool primitives, maximum flexibility via SKILL.md files
- 🔄 **Self-modifying**: The agent can improve itself by modifying its workspace
- 🔒 **Protected Core**: Immutable harness ensures stability while allowing agent evolution

:::

## Architecture Overview

```
┌─────────────────────────────────────┐
│        Interface (CLI / API)        │
└──────────────┬──────────────────────┘
               │
┌──────────────▼──────────────────────┐
│     Harness (Orchestrator-Loop)     │
│  System-Prompt ← SOUL.md + Skills  │
│  Tool-Execution + Session-Mgmt     │
└──────────────┬──────────────────────┘
               │
┌──────────────▼──────────────────────┐
│     Brain (Local LLM via Ollama)    │
│  Qwen3-30B-A3B / Tool-Calling      │
└──────────────┬──────────────────────┘
               │
┌──────────────▼──────────────────────┐
│   Tools + Skills + Memory           │
│  Files · Bash · Memory · Skills    │
│  (Agent can modify workspace)      │
└─────────────────────────────────────┘
```

## Quick Start

Get xatrix running in 3 steps:

```bash
# 1. Install Ollama and pull a model
ollama pull qwen3:30b-a3b
# Or use: gpt-oss:120b, llama3, mistral, etc.

# 2. Install xatrix
pip install -e .

# 3. Start the agent
xatrix
```

:::info Next Steps

- 📚 [Getting Started Guide](/getting-started) - Detailed installation and setup
- 🏗️ [Architecture](/architecture/overview) - Deep dive into how xatrix works
- 🔌 [Integrations](/integrations/whatsapp) - Connect xatrix to WhatsApp, Telegram, etc.

:::

## Core Concepts

### Skills

Skills are defined in `SKILL.md` files and teach the agent new capabilities without modifying code:

```markdown
# Code Assistant Skill

Help users write, debug, and refactor code.

## When to use
- User asks for code help
- User reports a bug
- User wants to refactor

## How to proceed
1. Understand the requirement
2. Check existing code context
3. Provide solution with explanation
```

### The Harness

The harness is the orchestrator that runs the agent loop:
- Loads skills and system prompt
- Manages tool execution
- Handles conversation flow
- Coordinates between brain and workspace

### Memory System

Persistent memory using SQLite allows the agent to remember:
- Past conversations
- User preferences
- Task context
- Learned patterns

## Project Status

- ✅ Core harness with tool-calling
- ✅ Skill system (SKILL.md)
- ✅ Memory (SQLite)
- ✅ CLI interface
- ✅ Job queue system
- ✅ WhatsApp integration (Baileys)
- 🚧 Container sandbox (Docker)
- 🚧 Voice interface (TTS/STT)
- 📋 Semantic memory search (Embeddings)
