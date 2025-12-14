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

### rails-code-review

Systematic technique for comprehensive Rails 8 code reviews covering security vulnerabilities (XSS, SQL injection), Rails conventions, Hotwire/Turbo patterns, performance optimization, and DRY principles with priority-based issue categorization.

**Use when:**
- Reviewing Rails code before merge or PR
- Performing security audits
- Validating Rails 8 conventions and Hotwire patterns

**Output:** Prioritized list of issues with location, problem, fix, and verification steps.

See [README.md](./skills/rails-code-review/README.md) for full documentation.

### rails-rspec-tester

Opinionated RSpec testing style for Rails applications emphasizing request specs, clean test organization, and DRY principles.

**Use when:**
- Writing new RSpec tests for Rails applications
- Converting controller tests to request specs
- Refactoring existing tests to follow best practices
- Reviewing test code for style compliance

**Key principles:**
- Request specs over controller tests (always)
- Setup in `let`/`let!` blocks, not in `it` blocks
- HTTP requests in `subject` blocks
- Context-driven organization
- Concise assertions

See [SKILL.md](./skills/rails-rspec-tester/SKILL.md) for full documentation.

### terraform-iac-expert

Senior staff-level DevOps expertise for Terraform and Infrastructure as Code across AWS, Azure, and GCP with 10+ years of FAANG experience.

**Use when:**
- Writing or reviewing Terraform configurations
- Designing cloud infrastructure
- Creating reusable Terraform modules
- Debugging Terraform state or deployment issues
- Setting up CI/CD pipelines for infrastructure
- Implementing security and compliance best practices

**Key principles:**
- Infrastructure as Code (IaC) first approach
- Modular, reusable, maintainable infrastructure
- Security by default with least privilege
- Multi-cloud expertise
- Environment isolation and workspace management

See [SKILL.md](./skills/terraform-iac-expert/SKILL.md) for full documentation.

## Agents

### rails-code-reviewer

Specialized subagent for comprehensive Rails 8 code reviews. Loads the rails-code-review skill and executes systematic review workflow.

**Use via:** Manual invocation or Task tool (when registered as subagent type)

## Commands

### /rails-code-review

One-command trigger for instant Rails code reviews.

**Usage:** `/rails-code-review` - Automatically loads skill and agent workflow

## System Architecture

The Rails code review system uses a three-layer architecture:
- **Command** (`/rails-code-review`) triggers
- **Agent** (`rails-code-reviewer`) which loads
- **Skill** (`rails-code-review`) technique guide

This ensures consistency, maintainability, and flexibility. See [README-rails-code-review-system.md](README-rails-code-review-system.md) for details.

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
