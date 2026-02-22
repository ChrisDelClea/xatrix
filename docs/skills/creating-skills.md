---
sidebar_position: 1
---

# Creating Skills

Skills are the primary way to extend your agent's capabilities. They're defined in simple Markdown files that describe **when** and **how** the agent should act.

## What is a Skill?

A skill is a `SKILL.md` file that teaches the agent a new capability without writing code. The agent reads these files and incorporates them into its behavior.

:::tip Philosophy
**Skills over Tools** - Instead of hard-coding every capability, we provide minimal tool primitives (read, write, bash, memory) and maximum flexibility via skills.
:::

## Skill Structure

Every `SKILL.md` file follows this template:

```markdown
# Skill Name

Brief description of what this skill does.

## When to use

- Condition 1 that triggers this skill
- Condition 2 that triggers this skill
- Example user request

## How to proceed

1. First step
2. Second step
3. Final step

## Examples

### Example 1: Specific scenario
User: "..."
Agent: "..." (explanation of what to do)

### Example 2: Another scenario
User: "..."
Agent: "..."
```

## Creating Your First Skill

Let's create a "Weather Checker" skill:

```bash
mkdir -p workspace/skills/weather_checker
cd workspace/skills/weather_checker
```

Create `SKILL.md`:

```markdown title="workspace/skills/weather_checker/SKILL.md"
# Weather Checker

Check current weather conditions for a location.

## When to use

- User asks about the weather
- User mentions weather-related terms (rain, sunny, temperature)
- User wants to know what to wear based on weather

## How to proceed

1. Extract the location from user's message
2. Use bash tool to curl wttr.in API: `curl wttr.in/CityName?format=3`
3. Parse the response and present in friendly format
4. If no location given, ask user for their location

## Examples

### Example 1: Direct weather request
User: "What's the weather in Berlin?"
Agent:
1. Extract location: "Berlin"
2. Run: `bash curl wttr.in/Berlin?format=3`
3. Parse result: "Berlin: ☀️ +22°C"
4. Respond: "It's sunny and 22°C in Berlin right now!"

### Example 2: No location given
User: "How's the weather?"
Agent: "I'd be happy to check the weather for you! Which city are you interested in?"
```

## Activating a Skill

Skills are loaded automatically from `workspace/skills/*/SKILL.md`. To activate:

1. Create the skill file
2. Restart xatrix or use `/reload` (if implemented)
3. The skill is now part of the agent's system prompt

Check loaded skills:

```bash
xatrix
> /skills
```

## Skill Best Practices

### ✅ Do's

- **Be specific** - Clear conditions for when to use the skill
- **Show examples** - Real scenarios help the LLM understand
- **Keep it focused** - One skill, one purpose
- **Use existing tools** - Leverage bash, read, write, memory tools
- **Test thoroughly** - Try edge cases and different phrasings

### ❌ Don'ts

- **Don't overlap** - Avoid duplicate skills with similar triggers
- **Don't be vague** - "Help users" is too general
- **Don't hard-code** - Use tools dynamically instead of fixed responses
- **Don't over-engineer** - Start simple, iterate

## Advanced Skill Patterns

### Multi-Step Workflow Skill

```markdown title="workspace/skills/code_reviewer/SKILL.md"
# Code Reviewer

Review code for best practices, bugs, and improvements.

## When to use

- User shares code for review
- User asks "review this code"
- User wants feedback on their implementation

## How to proceed

1. **Read the code** - Use grep/read tools to understand context
2. **Check for issues**:
   - Syntax errors
   - Logic bugs
   - Performance problems
   - Security vulnerabilities
3. **Suggest improvements**:
   - Better variable names
   - Refactoring opportunities
   - Best practices
4. **Provide examples** - Show improved code when suggesting changes

## Examples

### Example: User shares Python code
User: "Review this: def calc(x,y): return x+y"
Agent:
1. Analyze: Simple addition function, short names
2. Issues: Non-descriptive names, no type hints, no docstring
3. Suggest: Better naming, add types, add docstring
4. Provide improved version with explanation
```

### Contextual Memory Skill

```markdown title="workspace/skills/preference_tracker/SKILL.md"
# Preference Tracker

Remember and apply user preferences automatically.

## When to use

- User states a preference ("I prefer...", "I like...")
- User corrects you ("Actually, I use...")
- User asks "Do you remember...?"

## How to proceed

1. **Detect preference statements** in conversation
2. **Store in memory** using memory_store tool:
   - Key: "preference:topic"
   - Value: The preference
   - Example: memory_store("preference:language", "TypeScript over JavaScript")
3. **Recall when relevant** using memory_search
4. **Apply automatically** in future responses

## Examples

### Example: User states preference
User: "I prefer TypeScript over JavaScript"
Agent:
1. Detect preference statement
2. memory_store("preference:language", "TypeScript")
3. Confirm: "Got it! I'll use TypeScript in code examples for you."

### Example: Apply stored preference
User: "Can you help me build an API?"
Agent:
1. memory_search("preference:language") → "TypeScript"
2. Respond with TypeScript example instead of JavaScript
```

## Skill Composition

Skills can reference other skills:

```markdown
# Advanced Data Analysis

Combines multiple skills for comprehensive data analysis.

## Prerequisites

This skill builds on:
- `csv_parser` - For reading CSV files
- `chart_generator` - For creating visualizations
- `statistics_helper` - For calculating metrics

## When to use

- User uploads a dataset
- User asks for "data analysis"
- User wants insights from structured data
```

## Testing Your Skill

1. **Start xatrix** with your new skill
2. **Test trigger conditions**:
   ```
   You: What's the weather in Paris?
   Agent: [Should activate weather skill]
   ```
3. **Try edge cases**:
   - Missing information
   - Invalid inputs
   - Ambiguous requests
4. **Verify tool usage** - Check that correct tools are called
5. **Iterate** - Refine based on testing

## Debugging Skills

If your skill isn't working:

1. **Check skill is loaded**: `/skills` command
2. **Review system prompt**: Enable debug logging to see if skill appears
3. **Test conditions**: Are trigger conditions too specific/vague?
4. **Simplify**: Start with minimal version, add complexity gradually

## Next Steps

- [Architecture Overview](/architecture/overview) - Understand the system architecture
- [WhatsApp Integration](/integrations/whatsapp) - Connect your agent to WhatsApp
- [Getting Started](/getting-started) - Complete installation guide
