# Clay Guidelines Skill

Behavioral guidelines for writing production-quality code with discipline and clarity. Reduces common LLM coding mistakes through structured practices around assumptions, simplicity, scope control, and verification.

## Overview

This skill enforces patterns that prevent common failures when Claude writes code:
- **Overcomplication**: Building 1000 lines when 100 would suffice
- **Silent assumptions**: Guessing at requirements without confirming
- **Scope creep**: Refactoring adjacent code not related to the task
- **Missing verification**: Not defining clear success criteria

## Key Principles

1. **Surface assumptions explicitly** - Never silently fill in ambiguous requirements
2. **Manage confusion actively** - Stop and clarify before guessing
3. **Push back constructively** - Challenge bad ideas with alternatives
4. **Enforce simplicity** - Prefer boring, obvious solutions over clever ones
5. **Maintain scope discipline** - Touch only what's needed
6. **Clean up dead code** - But ask before deleting

## When to Use

This skill should be active when:
- Writing new code or features
- Refactoring existing code
- Debugging or fixing issues
- Reviewing code changes
- Architecting systems or APIs

The skill description includes these triggers for automatic activation.

## Skill Structure

```
clay-guidelines/
├── SKILL.md          # Main skill content with YAML frontmatter
└── README.md         # This file
```

## Integration with Claude

The skill includes proper YAML frontmatter for Claude's skills system:

```yaml
---
name: clay-guidelines
description: Behavioral guidelines to reduce common LLM coding mistakes. 
             Use when writing, reviewing, or refactoring code to avoid 
             overcomplication, make surgical changes, surface assumptions, 
             and define verifiable success criteria.
---
```

The description is designed to trigger when coding tasks are detected.

## What Makes This Skill Different

Unlike technical skills that teach "how to do X", this is a **behavioral skill** that teaches:
- How to communicate during coding
- How to handle uncertainty
- How to maintain discipline
- How to verify work

It's based on the principle: "You are the hands; the human is the architect."

## Key Sections in SKILL.md

1. **Critical Behaviors** (6 behaviors with clear priorities)
2. **Leverage Patterns** (4 patterns for better collaboration)
3. **Output Standards** (code quality + communication standards)
4. **Failure Modes to Avoid** (12 common mistakes)
5. **Quick Reference Checklist** (pre-completion verification)
6. **Example Application** (bad vs good approaches)

## Installation

To use this skill with Claude:

1. Package the skill directory
2. Upload to Claude (either via UI or appropriate installation method)
3. The skill will automatically activate based on the description triggers

## Example Usage

When Claude has this skill active:

**Without clay-guidelines:**
```
User: Add error handling to this function
Claude: [Adds comprehensive error handling with custom exceptions, 
logging framework integration, retry logic, and alerting]
```

**With clay-guidelines:**
```
User: Add error handling to this function
Claude:
ASSUMPTIONS I'M MAKING:
1. ValueError + logging is sufficient (not custom exception types)
2. Fail-fast strategy (no retries)
3. Log at ERROR level to existing logger
→ Correct me now or I'll proceed with these.

PLAN:
1. Add try/except around the risky call
2. Log the error with context
3. Re-raise with clear message
→ Executing unless you redirect.
```

## Customization

The skill can be customized by editing SKILL.md:

- Add domain-specific patterns to "Leverage Patterns"
- Modify priority levels for your team's needs
- Add examples from your codebase
- Adjust the checklist items

## Philosophy

The skill embodies a specific philosophy:
- **Correctness first, performance second**
- **Clarity over cleverness**
- **Surgical changes over renovations**
- **Explicit over implicit**
- **Question before implementing**

## Contributing

To improve this skill:
1. Note failure modes not covered
2. Add examples of good/bad approaches
3. Refine the assumption surfacing format
4. Add domain-specific guidance as needed

## License

This skill is based on the "Senior Software Engineer" system prompt pattern and adapted for Claude's skills framework.

## Related Skills

Works well with:
- Technical implementation skills (language-specific)
- Testing and verification skills
- Documentation and API design skills
- Code review skills

But clay-guidelines focuses on *behavior* and *communication*, not technical details.
