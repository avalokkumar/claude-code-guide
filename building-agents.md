# Building Agents with Claude Agent SDK

A complete guide to creating autonomous agents that can research, analyze, and take actions using the Claude Agent SDK.

---

## What Are Agents?

**Agents** are autonomous systems that use Claude to accomplish complex tasks by:

- Breaking down tasks into steps
- Using tools to gather information and take actions
- Maintaining context across multiple interactions
- Making decisions about what to do next

The **Claude Agent SDK** provides the infrastructure to build these agents efficiently.

---

## Getting Started

### Installation

```bash
# Install uv (Python package manager)
curl -LsSf https://astral.sh/uv/install.sh | sh

# Install Node.js and Claude Code CLI
npm install -g @anthropic-ai/claude-code

# Clone the cookbook and set up
git clone https://github.com/anthropics/anthropic-cookbook.git
cd anthropic-cookbook/claude_agent_sdk
uv sync
```

### Environment Setup

```bash
# Create .env file
echo 'ANTHROPIC_API_KEY=your-api-key-here' > .env

# Optional: GitHub token for observability agent
echo 'GITHUB_TOKEN=your-github-token' >> .env
```

---

## Agent Architecture

### Core Components

```
┌─────────────────────────────────────────────────────────┐
│                    Claude Agent SDK                      │
├─────────────────────────────────────────────────────────┤
│  ClaudeSDKClient                                         │
│  ├── query()           - Send prompts to agent          │
│  ├── receive_response() - Async stream of responses     │
│  └── Context management - Maintains conversation state   │
├─────────────────────────────────────────────────────────┤
│  ClaudeAgentOptions                                      │
│  ├── model             - Which Claude model to use      │
│  ├── allowed_tools     - Tools the agent can use        │
│  ├── system_prompt     - Agent's instructions           │
│  ├── permission_mode   - Execution vs planning mode     │
│  └── setting_sources   - Load project settings          │
├─────────────────────────────────────────────────────────┤
│  Available Tools                                         │
│  ├── Read              - Read files and images          │
│  ├── Write             - Create files                   │
│  ├── Edit              - Modify files                   │
│  ├── Bash              - Execute commands               │
│  ├── WebSearch         - Search the internet            │
│  └── Task              - Delegate to sub-agents         │
└─────────────────────────────────────────────────────────┘
```

---

## Example 1: Research Agent

A simple agent that searches the web and synthesizes information:

### agent.py

```python
"""
Research Agent - Web search and analysis capabilities.
"""

import asyncio
from claude_agent_sdk import ClaudeAgentOptions, ClaudeSDKClient
from dotenv import load_dotenv

load_dotenv()

SYSTEM_PROMPT = """You are a research agent specialized in gathering information.

When providing research findings:
- Always include source URLs as citations
- Format citations as markdown links: [Source Title](URL)
- Group sources in a "Sources:" section at the end
- Synthesize information, don't just list facts
"""

async def research(query: str, model: str = "claude-sonnet-4-5") -> str:
    """
    Research a topic using web search.
    
    Args:
        query: The research question
        model: Claude model to use
    
    Returns:
        Research findings with citations
    """
    
    options = ClaudeAgentOptions(
        model=model,
        allowed_tools=["WebSearch", "Read"],
        system_prompt=SYSTEM_PROMPT,
        max_buffer_size=10 * 1024 * 1024,  # 10MB for large responses
    )
    
    result = None
    
    async with ClaudeSDKClient(options=options) as agent:
        await agent.query(prompt=query)
        
        async for msg in agent.receive_response():
            # Print activity for visibility
            if hasattr(msg, 'content') and msg.content:
                first = msg.content[0] if isinstance(msg.content, list) else msg.content
                if hasattr(first, 'name'):
                    print(f"🔍 Using: {first.name}()")
            
            # Capture final result
            if hasattr(msg, 'result'):
                result = msg.result
    
    return result

# Run the agent
if __name__ == "__main__":
    query = "What are the latest developments in quantum computing in 2024?"
    result = asyncio.run(research(query))
    print("\n" + "="*50 + "\n")
    print(result)
```

### Usage

```bash
# Run the agent
python agent.py

# Or import in your code
from research_agent.agent import research

result = await research("What is the current state of AI regulation in the EU?")
```

---

## Example 2: Chief of Staff Agent

A sophisticated agent with sub-agents, custom scripts, and memory:

### Directory Structure

```
chief_of_staff_agent/
├── .claude/
│   ├── agents/                # Sub-agent definitions
│   │   ├── financial-analyst.md
│   │   └── recruiter.md
│   ├── commands/              # Slash commands
│   │   └── budget-impact.md
│   └── settings.local.json    # Hooks configuration
├── scripts/                   # Python utilities
│   ├── financial_forecast.py
│   └── decision_matrix.py
├── financial_data/            # Data files
│   └── q3_actuals.csv
├── CLAUDE.md                  # Agent context/memory
└── agent.py                   # Main agent code
```

### CLAUDE.md - Agent Memory

```markdown
# Chief of Staff Context

## Company Overview
- **Company**: TechStart Inc
- **Stage**: Series A ($10M raised)
- **Industry**: B2B SaaS - AI developer tools

## Financial Snapshot
- **Monthly Burn**: $500,000
- **Runway**: 20 months
- **ARR**: $2.4M (growing 15% MoM)

## Team Structure
- **Total**: 50 employees
- **Engineering**: 25 (50%)
- **Sales & Marketing**: 12 (24%)
- **Product**: 5 (10%)

## Current Priorities
1. Hire 10 engineers for Q2
2. Launch AI code review feature
3. Expand into European market
4. Begin Series B conversations

## Available Scripts
- `python scripts/financial_forecast.py` - Financial modeling
- `python scripts/decision_matrix.py` - Strategic decisions
```

### Sub-Agent Definition

```markdown
<!-- .claude/agents/financial-analyst.md -->
# Financial Analyst Agent

You are a financial analyst specializing in:
- Revenue forecasting
- Unit economics analysis
- Burn rate optimization
- Investor reporting

When analyzing data:
1. Always show your calculations
2. Compare to industry benchmarks
3. Highlight key risks
4. Provide actionable recommendations
```

### agent.py

```python
"""
Chief of Staff Agent - Multi-agent orchestration example.
"""

import asyncio
import json
import os
from typing import Literal

from claude_agent_sdk import ClaudeAgentOptions, ClaudeSDKClient
from dotenv import load_dotenv

load_dotenv()

SYSTEM_PROMPT = """You are the Chief of Staff for TechStart Inc.

You have access to:
- Financial data in financial_data/ directory
- Python scripts in scripts/ directory for analysis
- Two specialized sub-agents: financial-analyst and recruiter

When handling complex requests:
1. Break down the task
2. Delegate to appropriate sub-agents
3. Synthesize their findings
4. Provide executive-level recommendations
"""

async def send_query(
    prompt: str,
    continue_conversation: bool = False,
    permission_mode: Literal["default", "plan", "acceptEdits"] = "default",
    output_style: str | None = None,
) -> tuple[str | None, list]:
    """
    Send a query to the Chief of Staff agent.
    
    Args:
        prompt: The query (can include /commands)
        continue_conversation: Continue previous conversation
        permission_mode: "default", "plan" (think only), "acceptEdits"
        output_style: Override output style (e.g., "executive", "technical")
    
    Returns:
        Tuple of (result_text, all_messages)
    """
    
    settings = None
    if output_style:
        settings = json.dumps({"outputStyle": output_style})
    
    options = ClaudeAgentOptions(
        model="claude-opus-4-5",
        allowed_tools=[
            "Task",       # Sub-agent delegation
            "Read",       # File reading
            "Write",      # File creation
            "Edit",       # File editing
            "Bash",       # Script execution
            "WebSearch",  # Web research
        ],
        continue_conversation=continue_conversation,
        system_prompt=SYSTEM_PROMPT,
        permission_mode=permission_mode,
        cwd=os.path.dirname(os.path.abspath(__file__)),
        settings=settings,
        # Load project settings (CLAUDE.md, agents, commands, hooks)
        setting_sources=["project", "local"],
    )
    
    result = None
    messages = []
    
    async with ClaudeSDKClient(options=options) as agent:
        await agent.query(prompt=prompt)
        
        async for msg in agent.receive_response():
            messages.append(msg)
            
            # Activity logging
            if hasattr(msg, 'content') and msg.content:
                first = msg.content[0] if isinstance(msg.content, list) else msg.content
                if hasattr(first, 'name'):
                    print(f"🤖 Using: {first.name}()")
            
            if hasattr(msg, 'result'):
                result = msg.result
    
    return result, messages

# Example usage
if __name__ == "__main__":
    # Ask for hiring analysis with sub-agent delegation
    result, _ = asyncio.run(send_query(
        "Analyze our Q3 financials and recommend whether we should "
        "accelerate engineering hiring or invest in sales. "
        "Consider our runway and growth targets."
    ))
    print(result)
```

---

## Example 3: Observability Agent

An agent that monitors GitHub repos and CI/CD pipelines:

### agent.py

```python
"""
Observability Agent - DevOps monitoring with MCP integration.
"""

import asyncio
from claude_agent_sdk import ClaudeAgentOptions, ClaudeSDKClient
from dotenv import load_dotenv

load_dotenv()

SYSTEM_PROMPT = """You are a DevOps observability agent.

Your responsibilities:
- Monitor repository health
- Analyze CI/CD pipeline failures
- Investigate incidents
- Provide actionable insights

When investigating issues:
1. Gather relevant logs and data
2. Identify root cause
3. Suggest remediation steps
4. Note any patterns or recurring issues
"""

async def monitor_repo(repo: str, analysis_type: str = "health") -> str:
    """
    Monitor a GitHub repository.
    
    Args:
        repo: Repository in format "owner/repo"
        analysis_type: "health", "failures", "activity"
    """
    
    options = ClaudeAgentOptions(
        model="claude-sonnet-4-5",
        allowed_tools=["WebSearch", "Read", "Bash"],
        system_prompt=SYSTEM_PROMPT,
        # MCP servers for GitHub integration
        mcp_servers={
            "github": {
                "command": "mcp-server-github",
                "env": {"GITHUB_TOKEN": "${GITHUB_TOKEN}"}
            }
        }
    )
    
    prompts = {
        "health": f"Analyze the overall health of {repo}. Check recent commits, open issues, and CI status.",
        "failures": f"Investigate recent CI/CD failures in {repo}. Find patterns and root causes.",
        "activity": f"Summarize recent activity in {repo} including PRs, issues, and releases."
    }
    
    result = None
    
    async with ClaudeSDKClient(options=options) as agent:
        await agent.query(prompt=prompts.get(analysis_type, prompts["health"]))
        
        async for msg in agent.receive_response():
            if hasattr(msg, 'result'):
                result = msg.result
    
    return result
```

---

## Key SDK Concepts

### ClaudeAgentOptions

| Option | Type | Description |
|--------|------|-------------|
| `model` | str | Claude model (opus, sonnet, haiku) |
| `allowed_tools` | list | Tools the agent can use |
| `system_prompt` | str | Instructions for the agent |
| `continue_conversation` | bool | Maintain conversation state |
| `permission_mode` | str | "default", "plan", "acceptEdits" |
| `cwd` | str | Working directory |
| `settings` | str | JSON settings string |
| `setting_sources` | list | ["project", "local", "user"] |
| `max_buffer_size` | int | Buffer size for large responses |

### Available Tools

| Tool | Purpose | Example Use |
|------|---------|-------------|
| `Read` | Read files/images | Analyze code, view images |
| `Write` | Create files | Generate reports, save data |
| `Edit` | Modify files | Fix bugs, update configs |
| `Bash` | Run commands | Execute scripts, git ops |
| `WebSearch` | Internet search | Research, find docs |
| `Task` | Delegate to sub-agents | Complex multi-step tasks |

### Permission Modes

| Mode | Behavior |
|------|----------|
| `default` | Execute with user approval |
| `plan` | Plan only, no execution |
| `acceptEdits` | Auto-approve file changes |

---

## Sub-Agent Patterns

### Defining Sub-Agents

Create `.claude/agents/agent-name.md`:

```markdown
# Agent Name

Brief description of the agent's role.

## Expertise
- Domain 1
- Domain 2

## Instructions
When handling tasks:
1. Step one
2. Step two

## Constraints
- What not to do
- Limitations
```

### Delegating to Sub-Agents

```python
# In your main agent, include Task tool
options = ClaudeAgentOptions(
    allowed_tools=["Task", ...],  # Task enables sub-agent delegation
    setting_sources=["project"],   # Load agent definitions
)

# Claude will automatically use sub-agents when appropriate
await agent.query(
    "Analyze our financials (use financial-analyst) and "
    "evaluate the engineering candidate (use recruiter)"
)
```

---

## Hooks and Events

### Setting Up Hooks

Create `.claude/settings.local.json`:

```json
{
  "hooks": {
    "preCommit": ".claude/hooks/pre-commit.sh",
    "postEdit": ".claude/hooks/lint.sh",
    "onError": ".claude/hooks/notify.sh"
  }
}
```

### Hook Script Example

```bash
#!/bin/bash
# .claude/hooks/pre-commit.sh

echo "Running pre-commit checks..."
npm run lint
npm run test -- --bail
```

---

## Best Practices

### 1. Clear System Prompts

```python
# ❌ Vague
SYSTEM_PROMPT = "You are a helpful assistant."

# ✅ Specific
SYSTEM_PROMPT = """You are a security analyst for web applications.

Responsibilities:
- Review code for OWASP Top 10 vulnerabilities
- Check for hardcoded secrets
- Validate input sanitization

Output format:
1. Executive summary
2. Findings (severity: Critical/High/Medium/Low)
3. Recommendations
"""
```

### 2. Appropriate Tool Selection

```python
# Research agent - needs web access
options = ClaudeAgentOptions(
    allowed_tools=["WebSearch", "Read"],
)

# Code review agent - needs file access
options = ClaudeAgentOptions(
    allowed_tools=["Read", "Edit", "Bash"],
)

# Orchestrator agent - needs delegation
options = ClaudeAgentOptions(
    allowed_tools=["Task", "Read", "WebSearch"],
)
```

### 3. Error Handling

```python
async def robust_query(prompt: str) -> str | None:
    try:
        async with ClaudeSDKClient(options=options) as agent:
            await agent.query(prompt=prompt)
            async for msg in agent.receive_response():
                if hasattr(msg, 'result'):
                    return msg.result
    except Exception as e:
        print(f"❌ Agent error: {e}")
        # Log, retry, or escalate
        return None
```

### 4. Context Management

```python
# For multi-turn conversations
result1, _ = await send_query("What's our current ARR?")
result2, _ = await send_query(
    "How does that compare to last quarter?",
    continue_conversation=True  # Maintains context
)
```

---

## Running Agents

### As Scripts

```bash
python agent.py
```

### In Notebooks

```python
# Register Jupyter kernel
uv run python -m ipykernel install --user --name="agent-env"

# In notebook
import asyncio
from agent import research

result = await research("Latest AI safety research")
```

### As Modules

```bash
# Install as editable package
uv pip install -e .

# Import anywhere
from research_agent.agent import research
```

---

## Troubleshooting

| Issue | Solution |
|-------|----------|
| "API key not found" | Check .env file and load_dotenv() |
| "Tool not available" | Add tool to allowed_tools list |
| "Sub-agent not found" | Check .claude/agents/ and setting_sources |
| "Context too long" | Increase max_buffer_size |
| "Permission denied" | Check permission_mode setting |

---

## Related Documentation

- [Claude Code Features](./features.md) - Full feature overview
- [Building Skills](./building-skills.md) - Document generation
- [Quick Reference](./quick-reference.md) - Commands and snippets
