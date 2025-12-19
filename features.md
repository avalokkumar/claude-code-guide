# Claude Code Features Overview

A complete guide to all Claude Code capabilities and how to use them effectively.

---

## Core Features

### 1. Interactive Terminal Mode

Start Claude Code in interactive mode for ongoing conversations:

```bash
# Start interactive session
claude

# With specific working directory
claude --cwd /path/to/project

# With verbose output
claude --verbose
```

### 2. Single Query Mode

Run one-off queries without entering interactive mode:

```bash
# Simple query
claude "Explain this error: TypeError: undefined is not a function"

# Query with file context
claude "Review this code for bugs" --file src/app.js

# Query with multiple files
claude "Compare these implementations" --file v1.py --file v2.py
```

---

## Built-in Tools

Claude Code comes with powerful built-in tools:

### Read Tool

Read files, images, and documents:

```bash
# In interactive mode
> Read the contents of config.json and explain the settings

# Claude can read:
# - Text files (any extension)
# - Images (png, jpg, gif, webp)
# - PDFs (extracts text)
# - Code files with syntax understanding
```

### Write Tool

Create new files:

```bash
> Create a new Python file called utils.py with helper functions for string manipulation

# Claude will:
# 1. Generate the code
# 2. Ask for confirmation
# 3. Write the file
```

### Edit Tool

Modify existing files with precision:

```bash
> In app.js, change the API endpoint from /api/v1 to /api/v2

# Claude uses surgical edits, not full file rewrites
# Preserves formatting and unrelated code
```

### Bash Tool

Execute shell commands:

```bash
> Run the tests and show me any failures

# Claude can run:
# - Build commands (npm, pip, cargo, etc.)
# - Git commands
# - Test runners
# - Any shell command (with your approval)
```

### WebSearch Tool

Research topics online:

```bash
> Search for the latest React 19 features and summarize them

# Great for:
# - Finding documentation
# - Researching libraries
# - Getting current information
```

### Task Tool (Sub-agents)

Delegate work to specialized sub-agents:

```bash
> Use the financial-analyst agent to review our Q3 numbers

# Sub-agents can be:
# - Pre-defined in .claude/agents/
# - Specialized for specific domains
# - Given their own tools and context
```

---

## Configuration System

### CLAUDE.md - Project Instructions

Create a `CLAUDE.md` file in your project root to give Claude context:

```markdown
# Project: MyApp

## Overview
This is a Next.js application with TypeScript and Tailwind CSS.

## Code Style
- Use functional components with hooks
- Prefer named exports
- Use TypeScript strict mode

## Important Files
- src/lib/api.ts - API client
- src/hooks/ - Custom React hooks
- src/components/ui/ - Shared UI components

## Commands
- `npm run dev` - Start development server
- `npm run test` - Run tests
- `npm run build` - Production build

## Notes
- API keys are in .env.local (never commit!)
- Use the design system colors from tailwind.config.js
```

### .claude/ Directory Structure

```
.claude/
├── settings.json      # Local settings
├── settings.local.json # Git-ignored personal settings
├── commands/          # Custom slash commands
│   ├── review.md      # /review command
│   └── deploy.md      # /deploy command
├── agents/            # Sub-agent definitions
│   ├── analyst.md     # Financial analyst agent
│   └── reviewer.md    # Code review agent
├── hooks/             # Event hooks
│   └── pre-commit.sh  # Run before commits
└── output-styles/     # Custom output formats
    ├── executive.md   # Executive summary style
    └── technical.md   # Detailed technical style
```

### Custom Slash Commands

Create reusable commands in `.claude/commands/`:

```markdown
<!-- .claude/commands/review.md -->
# Code Review Command

Review the provided code for:
1. Security vulnerabilities
2. Performance issues
3. Code style violations
4. Potential bugs

Provide a summary with severity ratings (Critical/High/Medium/Low).
```

Use it:

```bash
> /review src/auth/login.ts
```

---

## Permission Modes

Control how Claude executes actions:

### Default Mode

Claude asks before making changes:

```bash
claude  # Normal interactive mode
```

### Plan Mode

Claude plans but doesn't execute:

```bash
# Claude will explain what it WOULD do without doing it
> (in plan mode) Refactor the authentication module
```

### Accept Edits Mode

Auto-approve file edits (careful!):

```bash
claude --accept-edits
```

---

## Output Styles

Customize how Claude formats responses:

### Built-in Styles

- **Default** - Balanced, conversational
- **Concise** - Brief, to the point
- **Detailed** - Comprehensive explanations

### Custom Output Styles

Create `.claude/output-styles/executive.md`:

```markdown
# Executive Output Style

Format all responses as:
1. **Summary** (2-3 sentences)
2. **Key Points** (bullet list)
3. **Recommendation** (if applicable)
4. **Next Steps** (actionable items)

Use business language, avoid technical jargon.
Keep total length under 200 words.
```

---

## Model Selection

Choose the right model for your task:

| Model | Best For | Speed | Cost |
|-------|----------|-------|------|
| `claude-opus-4-5` | Complex reasoning, research | Slower | $$$ |
| `claude-sonnet-4-5` | Balanced quality/speed | Medium | $$ |
| `claude-haiku-4-5` | Quick tasks, simple queries | Fast | $ |

```bash
# Use specific model
claude --model claude-sonnet-4-5
```

---

## Conversation Management

### Continue Conversations

```bash
# Continue last conversation
claude --continue

# Reference previous context
> Following up on our earlier discussion about the database schema...
```

### Reset Context

```bash
# Start fresh
claude --new

# Or in interactive mode
> /clear
```

---

## Integration Features

### MCP (Model Context Protocol)

Connect to external systems:

```json
// .claude/mcp.json
{
  "servers": {
    "github": {
      "command": "mcp-server-github",
      "env": {
        "GITHUB_TOKEN": "${GITHUB_TOKEN}"
      }
    },
    "postgres": {
      "command": "mcp-server-postgres",
      "env": {
        "DATABASE_URL": "${DATABASE_URL}"
      }
    }
  }
}
```

### Git Integration

```bash
# Claude understands git context
> What changed in the last 5 commits?
> Create a commit message for my staged changes
> Help me resolve merge conflicts in src/app.ts
```

### Environment Variables

```bash
# Required
ANTHROPIC_API_KEY=sk-ant-...

# Optional
ANTHROPIC_BASE_URL=https://api.anthropic.com  # Custom endpoint
CLAUDE_MODEL=claude-sonnet-4-5                  # Default model
```

---

## Advanced Features

### Hooks

Automate actions on events:

```json
// .claude/settings.local.json
{
  "hooks": {
    "preCommit": ".claude/hooks/pre-commit.sh",
    "postEdit": ".claude/hooks/format.sh"
  }
}
```

### Sub-agents

Define specialized agents in `.claude/agents/`:

```markdown
<!-- .claude/agents/security-reviewer.md -->
# Security Reviewer Agent

You are a security expert focused on:
- OWASP Top 10 vulnerabilities
- Authentication/authorization issues
- Data validation and sanitization
- Secrets management

When reviewing code, always check for:
1. SQL injection
2. XSS vulnerabilities
3. Hardcoded credentials
4. Insecure dependencies
```

---

## Tips for Effective Use

### 1. Be Specific

```bash
# ❌ Vague
> Fix the bug

# ✅ Specific
> Fix the null pointer exception in UserService.getUser() 
> when the user ID doesn't exist in the database
```

### 2. Provide Context

```bash
# ❌ No context
> Write a function to process data

# ✅ With context
> Write a TypeScript function that takes an array of user objects
> and returns only users who have been active in the last 30 days.
> Use the User type from src/types.ts
```

### 3. Use CLAUDE.md

Keep project context in `CLAUDE.md` so Claude always knows:
- Tech stack and conventions
- Important files and their purposes
- Common commands and workflows

### 4. Leverage Sub-agents

For complex tasks, create specialized agents:
- Financial analysis
- Security review
- Documentation writing
- Test generation

---

## Common Commands Quick Reference

```bash
# Start interactive mode
claude

# Single query
claude "your question"

# With file context
claude --file src/main.py "explain this"

# Continue conversation
claude --continue

# Use specific model
claude --model claude-opus-4-5

# Plan mode (no execution)
claude --plan

# Auto-approve edits
claude --accept-edits

# Verbose output
claude --verbose
```

---

## Related Documentation

- [Building Skills](./building-skills.md) - Create document generation skills
- [Building Agents](./building-agents.md) - Create autonomous agents
- [Quick Reference](./quick-reference.md) - Command cheat sheet
