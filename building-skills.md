# Building Skills for Claude

A complete guide to creating custom Skills that give Claude specialized document generation capabilities.

---

## What Are Skills?

**Skills** are organized packages of instructions, code, and resources that give Claude specialized capabilities. Think of them as "expertise packages" that Claude can load dynamically.

### Built-in Skills

| Skill | ID | Description |
|-------|-----|-------------|
| Excel | `xlsx` | Create spreadsheets with formulas, charts, formatting |
| PowerPoint | `pptx` | Generate presentations with slides, images, transitions |
| PDF | `pdf` | Create formatted documents with text, tables, images |
| Word | `docx` | Generate rich documents with styling and structure |

### Custom Skills

You can create your own skills for:
- Company brand guidelines
- Financial analysis templates
- Report generation workflows
- Domain-specific calculations
- Custom document formats

---

## Using Built-in Skills

### Basic API Setup

```python
from anthropic import Anthropic

# Client with Skills beta headers
client = Anthropic(api_key="your-api-key")

# Create a message with Skills enabled
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
        "content": "Create an Excel file with a monthly budget tracker"
    }]
)
```

### Downloading Generated Files

Skills generate files that you download via the Files API:

```python
from anthropic import Anthropic

client = Anthropic()

def extract_file_ids(response) -> list[str]:
    """Extract file IDs from skill response."""
    file_ids = []
    
    for block in response.content:
        if hasattr(block, 'content'):
            # Look for file_id in nested content
            for item in block.content if isinstance(block.content, list) else [block.content]:
                if hasattr(item, 'file_id'):
                    file_ids.append(item.file_id)
    
    return file_ids

def download_file(file_id: str, output_path: str):
    """Download a file from the Files API."""
    content = client.beta.files.download(file_id=file_id)
    
    with open(output_path, 'wb') as f:
        f.write(content.read())
    
    print(f"✅ Downloaded: {output_path}")

# After getting response from skills
file_ids = extract_file_ids(response)
for i, file_id in enumerate(file_ids):
    download_file(file_id, f"output_{i}.xlsx")
```

---

## Creating Custom Skills

### Skill Directory Structure

```
my_skill/
├── SKILL.md           # Required: Main instructions for Claude
├── REFERENCE.md       # Optional: Additional reference material
├── scripts/           # Optional: Python/JavaScript code
│   └── processor.py
└── resources/         # Optional: Templates, data files
    └── template.xlsx
```

### SKILL.md Format

Every skill needs a `SKILL.md` file with YAML frontmatter:

```markdown
---
name: my-custom-skill
description: Brief description of what this skill does (max 1024 chars)
---

# My Custom Skill

## Purpose
Explain what this skill enables Claude to do.

## Instructions
Detailed instructions for Claude on how to use this skill.

## Available Scripts
- `scripts/processor.py` - Description of what it does

## Available Resources
- `resources/template.xlsx` - Template file for output

## Usage Examples

### Example 1: Basic Usage
When asked to [do something], follow these steps:
1. Step one
2. Step two
3. Step three

### Example 2: Advanced Usage
For complex requests, consider:
- Option A
- Option B
```

---

## Example: Brand Guidelines Skill

A skill that ensures all documents follow company branding:

### Directory Structure

```
applying-brand-guidelines/
├── SKILL.md
├── scripts/
│   ├── apply_brand.py
│   └── validate_brand.py
└── resources/
    └── logo.png
```

### SKILL.md

```markdown
---
name: applying-brand-guidelines
description: Applies consistent corporate branding to all generated documents
---

# Corporate Brand Guidelines Skill

This skill ensures all generated documents adhere to corporate brand standards.

## Brand Identity

**Company**: Acme Corporation
**Tagline**: "Innovation Through Excellence"

## Visual Standards

### Color Palette

**Primary Colors**:
- **Acme Blue**: #0066CC - Headers, primary buttons
- **Acme Navy**: #003366 - Text, accents
- **White**: #FFFFFF - Backgrounds

**Secondary Colors**:
- **Success Green**: #28A745 - Positive metrics
- **Warning Amber**: #FFC107 - Cautions
- **Error Red**: #DC3545 - Negative values

### Typography

**Font Hierarchy**:
- **H1**: 32pt, Bold, Acme Blue
- **H2**: 24pt, Semibold, Acme Navy
- **Body**: 11pt, Regular, Acme Navy

## Document Standards

### Excel Spreadsheets
- Headers: Bold, White text on Acme Blue background
- Data cells: Regular, Acme Navy text
- Alternating rows: Light gray (#F8F9FA)

### PowerPoint Presentations
- Title slide: Company logo, presentation title, date
- Content slides: Blue header bar, white content area
- Maximum 6 bullet points per slide

### PDF Documents
- Header: Logo left, title center, page number right
- Margins: 1 inch all sides
- Main headings: Acme Blue, 16pt, bold

## Application Instructions

When creating any document:
1. Start with brand colors and fonts
2. Include logo on first page/slide
3. Use consistent formatting throughout
4. Review against brand standards
```

---

## Example: Financial Analyzer Skill

A skill for financial calculations and reporting:

### SKILL.md

```markdown
---
name: analyzing-financial-statements
description: Analyzes financial data and generates professional financial reports
---

# Financial Analyzer Skill

Expert financial analysis for business metrics and reporting.

## Capabilities

### Ratio Analysis
- **Liquidity**: Current ratio, quick ratio
- **Profitability**: Gross margin, net margin, ROE, ROA
- **Efficiency**: Asset turnover, inventory turnover
- **Leverage**: Debt-to-equity, interest coverage

### Calculations

#### Profitability Ratios
```
Gross Margin = (Revenue - COGS) / Revenue × 100
Net Margin = Net Income / Revenue × 100
ROE = Net Income / Shareholders Equity × 100
ROA = Net Income / Total Assets × 100
```

#### Liquidity Ratios
```
Current Ratio = Current Assets / Current Liabilities
Quick Ratio = (Current Assets - Inventory) / Current Liabilities
```

## Usage

When analyzing financial data:
1. Extract key figures from provided statements
2. Calculate relevant ratios
3. Compare to industry benchmarks
4. Provide insights and recommendations

## Output Format

Always structure analysis as:
1. **Executive Summary** - Key findings
2. **Financial Highlights** - Important metrics
3. **Ratio Analysis** - Detailed calculations
4. **Trends** - Period-over-period changes
5. **Recommendations** - Actionable insights
```

---

## Uploading Custom Skills

### Using the Skills API

```python
from anthropic import Anthropic
from anthropic.lib import files_from_dir

client = Anthropic()

def create_skill(skill_path: str, display_title: str) -> dict:
    """Create a new custom skill from a directory."""
    
    skill = client.beta.skills.create(
        display_title=display_title,
        files=files_from_dir(skill_path)
    )
    
    return {
        "skill_id": skill.id,
        "display_title": skill.display_title,
        "latest_version": skill.latest_version,
        "created_at": skill.created_at
    }

# Create the skill
result = create_skill(
    skill_path="./custom_skills/financial_analyzer",
    display_title="Financial Analyzer"
)
print(f"Created skill: {result['skill_id']}")
```

### Using Custom Skills

```python
# Use your custom skill alongside built-in skills
response = client.beta.messages.create(
    model="claude-sonnet-4-5-20250929",
    max_tokens=4096,
    betas=["code-execution-2025-08-25", "files-api-2025-04-14", "skills-2025-10-02"],
    container={
        "skills": [
            # Built-in Excel skill
            {"type": "anthropic", "skill_id": "xlsx", "version": "latest"},
            # Your custom skill
            {"type": "custom", "skill_id": "skill_abc123", "version": "latest"}
        ]
    },
    tools=[{"type": "code_execution_20250825", "name": "code_execution"}],
    messages=[{
        "role": "user",
        "content": "Create a financial report for Q3 with our brand styling"
    }]
)
```

---

## Managing Skills

### List Custom Skills

```python
def list_custom_skills(client: Anthropic) -> list:
    """List all custom skills."""
    
    skills_response = client.beta.skills.list(source="custom")
    
    for skill in skills_response.data:
        print(f"📦 {skill.display_title}")
        print(f"   ID: {skill.id}")
        print(f"   Version: {skill.latest_version}")
        print(f"   Created: {skill.created_at}")
    
    return skills_response.data
```

### Update a Skill

```python
def update_skill(client: Anthropic, skill_id: str, skill_path: str):
    """Create a new version of an existing skill."""
    
    version = client.beta.skills.versions.create(
        skill_id=skill_id,
        files=files_from_dir(skill_path)
    )
    
    print(f"✅ Created version {version.version} for skill {skill_id}")
    return version
```

### Delete a Skill

```python
def delete_skill(client: Anthropic, skill_id: str):
    """Delete a skill and all its versions."""
    
    # First delete all versions
    versions = client.beta.skills.versions.list(skill_id=skill_id)
    for version in versions.data:
        client.beta.skills.versions.delete(
            skill_id=skill_id, 
            version=version.version
        )
    
    # Then delete the skill
    client.beta.skills.delete(skill_id)
    print(f"✅ Deleted skill: {skill_id}")
```

---

## Skill Validation

Before uploading, validate your skill structure:

```python
from pathlib import Path

def validate_skill(skill_path: str) -> dict:
    """Validate a skill directory before upload."""
    
    result = {"valid": True, "errors": [], "warnings": []}
    skill_dir = Path(skill_path)
    
    # Check directory exists
    if not skill_dir.exists():
        result["valid"] = False
        result["errors"].append(f"Directory does not exist: {skill_path}")
        return result
    
    # Check for SKILL.md
    skill_md = skill_dir / "SKILL.md"
    if not skill_md.exists():
        result["valid"] = False
        result["errors"].append("SKILL.md file is required")
    else:
        content = skill_md.read_text()
        
        # Check YAML frontmatter
        if not content.startswith("---"):
            result["valid"] = False
            result["errors"].append("SKILL.md must start with YAML frontmatter (---)")
        else:
            # Check required fields
            if "name:" not in content:
                result["valid"] = False
                result["errors"].append("YAML must include 'name' field")
            if "description:" not in content:
                result["valid"] = False
                result["errors"].append("YAML must include 'description' field")
    
    # Check total size (max 8MB)
    total_size = sum(f.stat().st_size for f in skill_dir.rglob("*") if f.is_file())
    if total_size > 8 * 1024 * 1024:
        result["valid"] = False
        result["errors"].append(f"Total size exceeds 8MB ({total_size / 1024 / 1024:.2f} MB)")
    
    return result
```

---

## Best Practices

### 1. Clear Instructions

```markdown
# ❌ Vague
Use proper formatting.

# ✅ Specific
Format headers using:
- H1: 24pt, Bold, #003366
- H2: 18pt, Bold, #0066CC
- Body: 11pt, Regular, #333333
```

### 2. Include Examples

```markdown
## Usage Examples

### Creating a Budget Report
When asked to create a budget report:
1. Use the budget_template.xlsx as base
2. Apply brand colors to headers
3. Add variance formulas
4. Include summary charts
```

### 3. Define Boundaries

```markdown
## Limitations
This skill should NOT be used for:
- Legal documents (use legal-docs skill instead)
- External client communications without review
- Documents requiring signatures
```

### 4. Version Your Skills

```python
# Track versions for rollback capability
v1 = create_skill("./skill_v1", "My Skill")
# ... later ...
v2 = update_skill(client, v1["skill_id"], "./skill_v2")
```

---

## Troubleshooting

| Issue | Solution |
|-------|----------|
| "SKILL.md not found" | Ensure file exists in skill root directory |
| "Invalid YAML frontmatter" | Check for `---` delimiters and required fields |
| "Size exceeds limit" | Remove large files, keep under 8MB total |
| "Skill not loading" | Verify beta headers are included in request |
| "File not generated" | Check code_execution tool is included |

---

## Related Documentation

- [Claude Code Features](./features.md) - Full feature overview
- [Building Agents](./building-agents.md) - Create autonomous agents
- [Quick Reference](./quick-reference.md) - Commands and snippets
