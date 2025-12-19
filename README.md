# Claude Code Complete Guide

> A comprehensive guide to using Claude Code CLI, building Skills, and creating Agents.

## What is Claude Code?

**Claude Code** is Anthropic's CLI-based agentic coding assistant that runs in your terminal. It provides direct access to Claude's capabilities for:

- 🔧 **Coding assistance** - Write, debug, and refactor code
- 📁 **File operations** - Read, edit, and create files
- 🌐 **Web search** - Research topics and gather information
- 🤖 **Agent building** - Create autonomous agents for any task
- 📊 **Document generation** - Create Excel, PowerPoint, PDF files via Skills

## Documentation Index

| Document | Description |
|----------|-------------|
| [Features Overview](./features.md) | Complete list of Claude Code capabilities |
| [Building Skills](./building-skills.md) | Create custom skills for document generation |
| [Building Agents](./building-agents.md) | Create autonomous agents using the SDK |
| [Quick Reference](./quick-reference.md) | Commands and code snippets |
| [Examples](./examples.md) | Real-world use cases and templates |

## Quick Start

### 1. Install Claude Code

```bash
# Install Node.js first (if needed): https://nodejs.org/
npm install -g @anthropic-ai/claude-code
```

### 2. Configure API Key

```bash
# Set your Anthropic API key
export ANTHROPIC_API_KEY="your-api-key-here"

# Or add to your shell profile (~/.zshrc or ~/.bashrc)
echo 'export ANTHROPIC_API_KEY="your-api-key-here"' >> ~/.zshrc
```

### 3. Start Using Claude Code

```bash
# Start interactive mode
claude

# Or run a single query
claude "Explain what this code does" --file main.py
```

## Key Concepts

### Skills
**Skills** are packages of instructions and code that give Claude specialized capabilities like creating Excel files, PowerPoint presentations, or PDFs with your company's branding.

→ [Learn to build Skills](./building-skills.md)

### Agents
**Agents** are autonomous systems built with the Claude Agent SDK that can research, analyze, and take actions across multiple tools and data sources.

→ [Learn to build Agents](./building-agents.md)

### CLAUDE.md
A special file that provides context and instructions to Claude Code about your project. Place it in your project root to customize Claude's behavior.

### MCP (Model Context Protocol)
A standard for connecting Claude to external tools and data sources like GitHub, databases, and custom APIs.

## Architecture Overview

```
┌─────────────────────────────────────────────────────────┐
│                    Claude Code CLI                       │
├─────────────────────────────────────────────────────────┤
│  Built-in Tools                                          │
│  ├── Read     - Read files and images                   │
│  ├── Write    - Create new files                        │
│  ├── Edit     - Modify existing files                   │
│  ├── Bash     - Execute shell commands                  │
│  ├── WebSearch- Search the internet                     │
│  └── Task     - Delegate to sub-agents                  │
├─────────────────────────────────────────────────────────┤
│  Extensions                                              │
│  ├── Skills   - Document generation (xlsx, pptx, pdf)   │
│  ├── MCP      - External system integration             │
│  └── Hooks    - Custom event handlers                   │
├─────────────────────────────────────────────────────────┤
│  Configuration                                           │
│  ├── CLAUDE.md       - Project instructions             │
│  ├── .claude/        - Commands, agents, settings       │
│  └── settings.json   - User preferences                 │
└─────────────────────────────────────────────────────────┘
```

## Use Cases

| Category | Examples |
|----------|----------|
| **Development** | Code review, refactoring, debugging, documentation |
| **Research** | Web research, data analysis, report generation |
| **Automation** | File processing, data transformation, testing |
| **Documents** | Excel reports, presentations, PDFs |
| **DevOps** | CI/CD analysis, incident response, monitoring |

## Next Steps

1. **New to Claude Code?** → Start with [Features Overview](./features.md)
2. **Want to create documents?** → Read [Building Skills](./building-skills.md)
3. **Building automation?** → See [Building Agents](./building-agents.md)
4. **Need quick answers?** → Check [Quick Reference](./quick-reference.md)

---

## Related Documentation

- [Claude API Documentation](https://docs.anthropic.com)
- [Skills Documentation](https://docs.claude.com/en/docs/agents-and-tools/agent-skills/overview)
- [Claude Agent SDK](https://github.com/anthropics/claude-agent-sdk-python)
