# Agent Skills

Custom skills for Claude Code. Part of the [agent-marketplace](https://github.com/jtemplet/agent-marketplace).

## Installation

```bash
# Register marketplace
/plugin marketplace add jtemplet/agent-marketplace

# Install this plugin
/plugin install agent-skills@agent-marketplace
```

## Skills

### example-skill

A simple example skill demonstrating the skill format.

## Creating New Skills

Each skill should:
1. Live in its own directory under `skills/`
2. Have a `SKILL.md` file with YAML frontmatter
3. Follow the format shown in `skills/example-skill/SKILL.md`

### Skill Format

```markdown
---
name: skill-name
description: Brief description of when to use this skill
---

# Skill Name

## When to Use This Skill

[Describe triggering conditions...]

## The Process

[Step-by-step instructions...]
```

## License

MIT License
