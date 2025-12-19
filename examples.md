# Claude Code Examples

Real-world examples and templates for common use cases.

---

## Skills Examples

### 1. Create a Budget Spreadsheet

```python
from anthropic import Anthropic

client = Anthropic()

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
    messages=[{
        "role": "user",
        "content": """Create a monthly budget tracker Excel file with:
        
1. Income section (salary, freelance, other)
2. Expense categories (housing, food, transport, utilities, entertainment)
3. Monthly columns for Jan-Dec
4. Formulas for totals and savings
5. Conditional formatting: green for positive savings, red for negative
6. A summary chart showing income vs expenses"""
    }]
)

# Download the file
# (see quick-reference.md for download code)
```

### 2. Generate a Presentation

```python
response = client.beta.messages.create(
    model="claude-sonnet-4-5-20250929",
    max_tokens=4096,
    betas=["code-execution-2025-08-25", "files-api-2025-04-14", "skills-2025-10-02"],
    container={
        "skills": [
            {"type": "anthropic", "skill_id": "pptx", "version": "latest"}
        ]
    },
    tools=[{"type": "code_execution_20250825", "name": "code_execution"}],
    messages=[{
        "role": "user",
        "content": """Create a 5-slide PowerPoint presentation for a startup pitch:

Slide 1: Title - "AI-Powered Code Review" with company name "CodeSense"
Slide 2: Problem - Current code review challenges
Slide 3: Solution - Our AI approach with key features
Slide 4: Market - $5B market opportunity
Slide 5: Ask - $2M seed round, team backgrounds

Use a professional blue color scheme."""
    }]
)
```

### 3. Create PDF Report

```python
response = client.beta.messages.create(
    model="claude-sonnet-4-5-20250929",
    max_tokens=4096,
    betas=["code-execution-2025-08-25", "files-api-2025-04-14", "skills-2025-10-02"],
    container={
        "skills": [
            {"type": "anthropic", "skill_id": "pdf", "version": "latest"}
        ]
    },
    tools=[{"type": "code_execution_20250825", "name": "code_execution"}],
    messages=[{
        "role": "user",
        "content": """Create a professional quarterly report PDF with:

Executive Summary
- Q3 2024 highlights
- Key metrics: $2.4M ARR, 120 customers, 15% MoM growth

Financial Performance
- Revenue breakdown by segment
- Expense analysis

Operational Highlights
- Product launches
- Team growth

Outlook for Q4
- Goals and targets
- Risk factors"""
    }]
)
```

### 4. Multi-Skill Workflow

```python
# Create analysis in Excel, then presentation summarizing it
response = client.beta.messages.create(
    model="claude-sonnet-4-5-20250929",
    max_tokens=8192,
    betas=["code-execution-2025-08-25", "files-api-2025-04-14", "skills-2025-10-02"],
    container={
        "skills": [
            {"type": "anthropic", "skill_id": "xlsx", "version": "latest"},
            {"type": "anthropic", "skill_id": "pptx", "version": "latest"}
        ]
    },
    tools=[{"type": "code_execution_20250825", "name": "code_execution"}],
    messages=[{
        "role": "user",
        "content": """Given this sales data:
        
Q1: $500K, Q2: $650K, Q3: $800K, Q4: $950K
Product A: 40%, Product B: 35%, Product C: 25%

Create:
1. An Excel file with the data, quarterly growth calculations, and charts
2. A 3-slide PowerPoint summarizing the key insights for the board"""
    }]
)
```

---

## Agent Examples

### 1. Simple Research Agent

```python
import asyncio
from claude_agent_sdk import ClaudeAgentOptions, ClaudeSDKClient

async def research(topic: str) -> str:
    """Research a topic and return findings with citations."""
    
    options = ClaudeAgentOptions(
        model="claude-sonnet-4-5",
        allowed_tools=["WebSearch", "Read"],
        system_prompt="""You are a research assistant.
Always include source URLs as citations.
Format: [Source Title](URL)
End with a Sources section."""
    )
    
    result = None
    async with ClaudeSDKClient(options=options) as agent:
        await agent.query(prompt=f"Research: {topic}")
        async for msg in agent.receive_response():
            if hasattr(msg, 'result'):
                result = msg.result
    
    return result

# Usage
result = asyncio.run(research("Latest developments in quantum computing 2024"))
print(result)
```

### 2. Code Review Agent

```python
async def code_review(file_path: str) -> str:
    """Review code for bugs, security issues, and style."""
    
    options = ClaudeAgentOptions(
        model="claude-sonnet-4-5",
        allowed_tools=["Read", "Bash"],
        system_prompt="""You are a senior code reviewer.

Check for:
1. Security vulnerabilities (OWASP Top 10)
2. Performance issues
3. Code style violations
4. Potential bugs
5. Test coverage gaps

Output format:
## Critical Issues
## High Priority
## Medium Priority
## Suggestions
## Summary"""
    )
    
    result = None
    async with ClaudeSDKClient(options=options) as agent:
        await agent.query(prompt=f"Review the code in {file_path}")
        async for msg in agent.receive_response():
            if hasattr(msg, 'result'):
                result = msg.result
    
    return result

# Usage
result = asyncio.run(code_review("src/auth/login.py"))
```

### 3. Documentation Agent

```python
async def generate_docs(project_path: str) -> str:
    """Generate documentation for a project."""
    
    options = ClaudeAgentOptions(
        model="claude-sonnet-4-5",
        allowed_tools=["Read", "Write", "Bash"],
        system_prompt="""You are a technical documentation specialist.

For each code file, generate:
1. Module overview
2. Function/class documentation
3. Usage examples
4. API reference

Write output to docs/ directory in Markdown format."""
    )
    
    result = None
    async with ClaudeSDKClient(options=options) as agent:
        await agent.query(
            prompt=f"Generate comprehensive documentation for {project_path}"
        )
        async for msg in agent.receive_response():
            if hasattr(msg, 'result'):
                result = msg.result
    
    return result
```

### 4. Test Generator Agent

```python
async def generate_tests(file_path: str, framework: str = "pytest") -> str:
    """Generate unit tests for a code file."""
    
    options = ClaudeAgentOptions(
        model="claude-sonnet-4-5",
        allowed_tools=["Read", "Write"],
        system_prompt=f"""You are a QA engineer specializing in {framework}.

For each function/method:
1. Write happy path tests
2. Write edge case tests
3. Write error handling tests
4. Mock external dependencies

Use descriptive test names and include docstrings."""
    )
    
    result = None
    async with ClaudeSDKClient(options=options) as agent:
        await agent.query(
            prompt=f"Generate comprehensive {framework} tests for {file_path}"
        )
        async for msg in agent.receive_response():
            if hasattr(msg, 'result'):
                result = msg.result
    
    return result
```

### 5. Multi-Agent System

```python
import os

# Directory structure
"""
project/
├── .claude/
│   └── agents/
│       ├── analyst.md
│       └── writer.md
├── CLAUDE.md
└── agent.py
"""

# .claude/agents/analyst.md
"""
# Data Analyst Agent

You analyze data and provide insights.

Responsibilities:
- Statistical analysis
- Trend identification
- Anomaly detection

Output: Structured findings with confidence levels.
"""

# .claude/agents/writer.md
"""
# Report Writer Agent

You create professional reports from analysis.

Style:
- Executive-friendly language
- Visual emphasis on key metrics
- Actionable recommendations
"""

# Main orchestrator agent
async def analyze_and_report(data_path: str) -> str:
    """Orchestrate analysis and report generation."""
    
    options = ClaudeAgentOptions(
        model="claude-opus-4-5",
        allowed_tools=["Task", "Read", "Write"],
        system_prompt="""You are a project coordinator.

For analysis requests:
1. Delegate data analysis to the analyst agent
2. Delegate report writing to the writer agent
3. Synthesize their outputs
4. Provide final recommendations""",
        cwd=os.path.dirname(os.path.abspath(__file__)),
        setting_sources=["project"],  # Load agent definitions
    )
    
    result = None
    async with ClaudeSDKClient(options=options) as agent:
        await agent.query(
            prompt=f"""Analyze the data in {data_path} and create an executive report.

Use the analyst agent for deep data analysis.
Use the writer agent for report creation."""
        )
        async for msg in agent.receive_response():
            if hasattr(msg, 'result'):
                result = msg.result
    
    return result
```

---

## CLAUDE.md Examples

### For a Web Application

```markdown
# MyApp - React + Node.js Application

## Tech Stack
- Frontend: React 18, TypeScript, Tailwind CSS
- Backend: Node.js, Express, PostgreSQL
- Testing: Jest, Cypress

## Code Conventions
- Use functional components with hooks
- Prefer named exports
- Use TypeScript strict mode
- Follow Airbnb style guide

## Directory Structure
- `src/client/` - React frontend
- `src/server/` - Express backend
- `src/shared/` - Shared types and utilities

## Commands
- `npm run dev` - Start both servers
- `npm run test` - Run Jest tests
- `npm run e2e` - Run Cypress tests
- `npm run lint` - ESLint + Prettier

## Environment Variables
- `DATABASE_URL` - PostgreSQL connection
- `JWT_SECRET` - Auth token signing
- `REDIS_URL` - Session storage

## Important Notes
- Never commit .env files
- Run migrations before starting: `npm run migrate`
- Check src/types/ for shared interfaces
```

### For a Python ML Project

```markdown
# MLProject - Machine Learning Pipeline

## Environment
- Python 3.11+
- Poetry for dependency management
- PyTorch 2.0

## Structure
- `src/data/` - Data loading and preprocessing
- `src/models/` - Model definitions
- `src/training/` - Training loops
- `src/evaluation/` - Metrics and visualization
- `notebooks/` - Experiments

## Commands
- `poetry install` - Install dependencies
- `poetry run train` - Run training
- `poetry run evaluate` - Run evaluation
- `poetry run jupyter` - Start notebooks

## Code Style
- Type hints on all functions
- Docstrings in NumPy format
- Black + isort formatting

## Data
- Raw data in `data/raw/`
- Processed data in `data/processed/`
- Models saved to `models/`

## GPU Notes
- Default: Single GPU training
- Multi-GPU: Use `--distributed` flag
- Memory: Batch size 32 requires ~8GB VRAM
```

---

## Custom Skill Examples

### Brand Guidelines Skill

```markdown
---
name: company-brand
description: Applies Acme Corp branding to all documents
---

# Acme Corp Brand Guidelines

## Colors
- Primary: #0066CC (Acme Blue)
- Secondary: #003366 (Navy)
- Accent: #28A745 (Success Green)
- Background: #FFFFFF

## Typography
- Headers: Segoe UI, 24pt, Bold
- Body: Segoe UI, 11pt, Regular
- Code: Consolas, 10pt

## Logo Usage
- Position: Top-left
- Size: 120px width
- Clear space: 20px minimum

## Document Standards

### Excel
- Header row: White on Acme Blue
- Borders: Light gray
- Alternating rows: #F8F9FA

### PowerPoint
- Title slide: Centered logo
- Content: Blue header bar
- Charts: Brand colors only

### PDF
- Header: Logo left, title center
- Footer: Copyright and page number
- Margins: 1 inch all sides
```

### Financial Calculator Skill

```markdown
---
name: financial-calculator
description: Calculates financial ratios and metrics
---

# Financial Calculator Skill

## Available Calculations

### Profitability
- Gross Margin = (Revenue - COGS) / Revenue
- Net Margin = Net Income / Revenue
- ROE = Net Income / Equity
- ROA = Net Income / Assets

### Liquidity
- Current Ratio = Current Assets / Current Liabilities
- Quick Ratio = (Current Assets - Inventory) / Current Liabilities

### Valuation
- P/E Ratio = Price / EPS
- EV/EBITDA = Enterprise Value / EBITDA
- Price/Book = Price / Book Value

## Usage

When calculating metrics:
1. Request the necessary input values
2. Show the formula used
3. Provide the calculated result
4. Include industry benchmark comparison if available

## Output Format

| Metric | Value | Industry Avg | Status |
|--------|-------|--------------|--------|
| [name] | [val] | [benchmark]  | ✅/⚠️/❌ |
```

---

## Terminal Workflow Examples

### Daily Development Workflow

```bash
# Start your day
claude --cwd ~/projects/myapp

# Review yesterday's changes
> What changed in yesterday's commits? Summarize the key updates.

# Check for issues
> Are there any open issues assigned to me on GitHub?

# Start coding with context
> I'm working on the authentication module. What's the current state?

# Get help with a specific task
> Help me implement rate limiting for the login endpoint
```

### Code Review Workflow

```bash
# Review a PR
> Review the changes in PR #42 for security and performance issues

# Generate suggestions
> Suggest improvements for the error handling in src/api/

# Update documentation
> Update the README to reflect the new API endpoints
```

### Research and Documentation

```bash
# Research a topic
> Research the latest best practices for React state management in 2024

# Generate docs
> Generate API documentation for src/routes/

# Create a guide
> Write a getting started guide for new developers joining the project
```

---

## Related Documentation

- [Features Overview](./features.md) - All Claude Code capabilities
- [Building Skills](./building-skills.md) - Create custom skills
- [Building Agents](./building-agents.md) - Create autonomous agents
- [Quick Reference](./quick-reference.md) - Commands and snippets
