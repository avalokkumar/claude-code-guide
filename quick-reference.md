# Claude Code Quick Reference

Copy-paste ready commands, code snippets, and templates.

---

## CLI Commands

### Basic Usage

```bash
# Start interactive mode
claude

# Single query
claude "your question here"

# Query with file context
claude "explain this" --file main.py

# Continue last conversation
claude --continue

# Start fresh conversation
claude --new
```

### Configuration

```bash
# Use specific model
claude --model claude-opus-4-5
claude --model claude-sonnet-4-5
claude --model claude-haiku-4-5

# Set working directory
claude --cwd /path/to/project

# Verbose output
claude --verbose

# Plan mode (no execution)
claude --plan

# Auto-approve edits (careful!)
claude --accept-edits
```

### Environment Variables

```bash
# Required
export ANTHROPIC_API_KEY="sk-ant-..."

# Optional
export CLAUDE_MODEL="claude-sonnet-4-5"
export ANTHROPIC_BASE_URL="https://api.anthropic.com"
```

---

## Skills API

### Basic Setup

```python
from anthropic import Anthropic

client = Anthropic()

# Message with Skills enabled
response = client.beta.messages.create(
    model="claude-sonnet-4-5-20250929",
    max_tokens=4096,
    betas=["code-execution-2025-08-25", "files-api-2025-04-14", "skills-2025-10-02"],
    container={
        "skills": [
            {"type": "anthropic", "skill_id": "xlsx", "version": "latest"}
        ]
    },
    tools=[{"type": "code_execution_20250825", "name": "code_execution"}],
    messages=[{"role": "user", "content": "Create an Excel budget"}]
)
```

### Built-in Skills

```python
# Excel
{"type": "anthropic", "skill_id": "xlsx", "version": "latest"}

# PowerPoint
{"type": "anthropic", "skill_id": "pptx", "version": "latest"}

# PDF
{"type": "anthropic", "skill_id": "pdf", "version": "latest"}

# Word
{"type": "anthropic", "skill_id": "docx", "version": "latest"}
```

### Download Generated Files

```python
# Extract file IDs from response
def extract_file_ids(response):
    file_ids = []
    for block in response.content:
        if hasattr(block, 'content'):
            for item in (block.content if isinstance(block.content, list) else [block.content]):
                if hasattr(item, 'file_id'):
                    file_ids.append(item.file_id)
    return file_ids

# Download file
file_ids = extract_file_ids(response)
for fid in file_ids:
    content = client.beta.files.download(file_id=fid)
    with open(f"output.xlsx", "wb") as f:
        f.write(content.read())
```

### Create Custom Skill

```python
from anthropic.lib import files_from_dir

# Create skill
skill = client.beta.skills.create(
    display_title="My Skill",
    files=files_from_dir("./my_skill/")
)

# Use custom skill
{"type": "custom", "skill_id": skill.id, "version": "latest"}
```

---

## Agent SDK

### Basic Agent

```python
import asyncio
from claude_agent_sdk import ClaudeAgentOptions, ClaudeSDKClient

async def simple_agent(prompt: str) -> str:
    options = ClaudeAgentOptions(
        model="claude-sonnet-4-5",
        allowed_tools=["WebSearch", "Read"],
    )
    
    result = None
    async with ClaudeSDKClient(options=options) as agent:
        await agent.query(prompt=prompt)
        async for msg in agent.receive_response():
            if hasattr(msg, 'result'):
                result = msg.result
    
    return result

# Run
result = asyncio.run(simple_agent("Research quantum computing"))
```

### Agent with All Options

```python
options = ClaudeAgentOptions(
    model="claude-opus-4-5",
    allowed_tools=["Task", "Read", "Write", "Edit", "Bash", "WebSearch"],
    system_prompt="You are an expert assistant...",
    continue_conversation=False,
    permission_mode="default",  # or "plan", "acceptEdits"
    cwd="/path/to/project",
    settings=json.dumps({"outputStyle": "executive"}),
    setting_sources=["project", "local"],
    max_buffer_size=10 * 1024 * 1024,
)
```

### Continue Conversation

```python
# First query
options = ClaudeAgentOptions(...)
async with ClaudeSDKClient(options=options) as agent:
    await agent.query("What is our ARR?")
    # ... get result

# Follow-up (same options object)
options.continue_conversation = True
async with ClaudeSDKClient(options=options) as agent:
    await agent.query("How does that compare to Q2?")
    # ... get result
```

---

## Project Configuration

### CLAUDE.md Template

```markdown
# Project Name

## Overview
Brief description of the project.

## Tech Stack
- Language: Python 3.11
- Framework: FastAPI
- Database: PostgreSQL

## Code Style
- Follow PEP 8
- Use type hints
- Prefer composition over inheritance

## Important Files
- src/main.py - Application entry point
- src/api/ - API routes
- src/models/ - Data models

## Commands
- `make dev` - Start development server
- `make test` - Run tests
- `make lint` - Run linter

## Notes
- API keys in .env (never commit!)
- See docs/ for architecture details
```

### Custom Slash Command

```markdown
<!-- .claude/commands/review.md -->
# Code Review

Review the code for:
1. Security issues
2. Performance problems
3. Code style violations
4. Potential bugs

Output format:
- **Critical**: Issues that must be fixed
- **High**: Should be fixed soon
- **Medium**: Nice to have
- **Low**: Minor suggestions
```

### Sub-Agent Definition

```markdown
<!-- .claude/agents/analyst.md -->
# Financial Analyst

You specialize in:
- Revenue analysis
- Unit economics
- Financial modeling

When analyzing data:
1. Show calculations
2. Compare to benchmarks
3. Highlight risks
4. Provide recommendations
```

### Hooks Configuration

```json
// .claude/settings.local.json
{
  "hooks": {
    "preCommit": ".claude/hooks/pre-commit.sh",
    "postEdit": ".claude/hooks/format.sh"
  }
}
```

---

## SKILL.md Template

```markdown
---
name: my-skill-name
description: What this skill does (max 1024 chars)
---

# Skill Name

## Purpose
What this skill enables Claude to do.

## Instructions
Step-by-step guide for Claude.

## Available Scripts
- `scripts/helper.py` - Description

## Available Resources
- `resources/template.xlsx` - Template file

## Usage Examples

### Example 1
When asked to do X, follow these steps...
```

---

## File Structure Templates

### Skills Project

```
my_skill/
├── SKILL.md              # Required
├── REFERENCE.md          # Optional extra docs
├── scripts/              # Optional Python/JS
│   └── helper.py
└── resources/            # Optional templates
    └── template.xlsx
```

### Agent Project

```
my_agent/
├── .claude/
│   ├── agents/           # Sub-agents
│   ├── commands/         # Slash commands
│   └── settings.local.json
├── scripts/              # Utility scripts
├── data/                 # Data files
├── CLAUDE.md             # Agent context
└── agent.py              # Main agent
```

---

## Common Patterns

### Error Handling

```python
async def safe_query(prompt: str) -> str | None:
    try:
        async with ClaudeSDKClient(options=options) as agent:
            await agent.query(prompt=prompt)
            async for msg in agent.receive_response():
                if hasattr(msg, 'result'):
                    return msg.result
    except Exception as e:
        print(f"Error: {e}")
        return None
```

### Activity Logging

```python
async for msg in agent.receive_response():
    # Log tool usage
    if hasattr(msg, 'content') and msg.content:
        first = msg.content[0] if isinstance(msg.content, list) else msg.content
        if hasattr(first, 'name'):
            print(f"🔧 Using: {first.name}()")
    
    if hasattr(msg, 'result'):
        result = msg.result
```

### Multiple Skills

```python
container={
    "skills": [
        {"type": "anthropic", "skill_id": "xlsx", "version": "latest"},
        {"type": "anthropic", "skill_id": "pptx", "version": "latest"},
        {"type": "custom", "skill_id": "my_skill_id", "version": "latest"}
    ]
}
```

---

## Troubleshooting Quick Fixes

| Problem | Solution |
|---------|----------|
| API key error | `export ANTHROPIC_API_KEY=...` |
| Skills not working | Add beta headers and code_execution tool |
| File download fails | Use `.read()` not `.content` |
| Agent context lost | Set `continue_conversation=True` |
| Sub-agent not found | Add `"project"` to `setting_sources` |
| Tool not available | Add to `allowed_tools` list |

---

## Links

- [Anthropic API Docs](https://docs.anthropic.com)
- [Skills Documentation](https://docs.claude.com/en/docs/agents-and-tools/agent-skills/overview)
- [Claude Agent SDK](https://github.com/anthropics/claude-agent-sdk-python)
- [Claude Cookbooks](https://github.com/anthropics/claude-cookbooks)
