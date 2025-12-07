# Rails Code Review System

A comprehensive, multi-layered system for Rails 8 code reviews using skills, agents, and commands.

## Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                     /rails-code-review                          │
│                     (Slash Command)                             │
│                           │                                      │
│                           │ triggers                             │
│                           ▼                                      │
│              ┌────────────────────────┐                         │
│              │  rails-code-reviewer   │                         │
│              │       (Agent)          │                         │
│              └────────────────────────┘                         │
│                           │                                      │
│                           │ loads & follows                      │
│                           ▼                                      │
│              ┌────────────────────────┐                         │
│              │  rails-code-review     │                         │
│              │      (Skill)           │                         │
│              └────────────────────────┘                         │
└─────────────────────────────────────────────────────────────────┘
```

## Components

### 1. Skill: `rails-code-review`
**Plugin Path:** `skills/rails-code-review/SKILL.md`
**Reference:** Use `Skill` tool with name `rails-code-review`

**Purpose:** The foundational technique guide for Rails code reviews.

**Contains:**
- Systematic review methodology
- Priority-based checklists (Security → Conventions → Performance → DRY → Logic)
- Quick reference tables with red flags
- Before/after code examples
- Common mistakes and how to avoid them
- Output format specifications

**Key Features:**
- CSO-optimized for discoverability
- Follows `writing-skills` best practices
- Tested and proven effective
- Rails 8 and Hotwire-specific

**Use directly when:** You need to manually perform a Rails code review

### 2. Agent: `rails-code-reviewer`
**Plugin Path:** `agents/rails-code-reviewer.md`
**Reference:** Follow agent workflow or use Task tool (when registered)

**Purpose:** Specialized subagent that embodies Rails code review expertise.

**Workflow:**
1. Get git diff context
2. Load rails-code-review skill
3. Execute systematic review in priority order
4. Document issues with actionable fixes
5. Provide summary and recommendation

**Critical Rules:**
- Always load the skill first
- Review in priority order
- Provide concrete code fixes
- Include verification steps
- Note positive findings

**Quality Checklist:**
- Every issue has: Category, Priority, Location, Problem, Why, Fix, Verification
- Code examples (before/after) included
- Summary with counts and assessment provided
- Clear merge/fix recommendation given

**Use when:** You need a fresh Claude instance to perform comprehensive Rails review (via Task tool or manual invocation)

### 3. Command: `/rails-code-review`
**Plugin Path:** `commands/rails-code-review.md`
**Invocation:** `/rails-code-review` (when plugin installed)

**Purpose:** One-command trigger for Rails code reviews.

**Invocation:**
```bash
/rails-code-review
```

**What it does:**
1. Loads the rails-code-reviewer agent instructions
2. Triggers the agent's workflow
3. Ensures skill is followed exactly
4. Enforces output format

**Use when:** You want instant Rails code review without manual agent dispatch

## Usage Examples

### Quick Review via Slash Command
```markdown
User: /rails-code-review

Claude: [Loads agent → Loads skill → Reviews code → Provides comprehensive report]
```

### Manual Agent Invocation
```markdown
User: Follow the rails-code-reviewer agent to review my current branch

Claude: [Reads agent file → Follows workflow → Uses skill → Provides report]
```

### Direct Skill Usage
```markdown
User: Use the rails-code-review skill to check this controller for security issues

Claude: [Loads skill → Applies security checklist → Reports findings]
```

### Future: Task Tool Dispatch (when agent is registered)
```python
Task(
  subagent_type="rails-code-reviewer",
  description="Review Rails changes",
  prompt="Review feature/auth branch against main for security issues"
)
```

## Review Process Flow

```
1. Trigger (command/agent/skill)
          ↓
2. Load rails-code-review skill
          ↓
3. Get git diff: current branch vs main
          ↓
4. Security Review (CRITICAL)
   - XSS vulnerabilities
   - SQL injection
   - Mass assignment
   - Unsafe rendering
          ↓
5. Rails Conventions Review (HIGH)
   - Rails 8 Way patterns
   - Hotwire usage
   - RESTful conventions
          ↓
6. Performance Review (HIGH)
   - N+1 queries
   - Missing indexes
   - Caching opportunities
          ↓
7. Code Duplication Review (MEDIUM)
   - Extract to partials
   - Shared helpers
   - DRY violations
          ↓
8. CSS/Styling Review (LOW)
   - Tailwind v4 syntax
   - Redundant classes
          ↓
9. Bug & Logic Review (ALL)
   - Logic errors
   - Missing validations
   - Edge cases
          ↓
10. Generate Report
    - Issues by priority
    - Critical actions
    - Positive findings
    - Merge recommendation
```

## Output Format (Standardized Across All Three)

```markdown
# Rails Code Review Report

**Branch:** feature/add-authentication
**Base:** main
**Files Changed:** 12

---

## [Category - PRIORITY] Issue Title

**Location:** `app/controllers/users_controller.rb:45`

**Problem:**
[Description with code snippet]

**Why it matters:**
[Impact explanation]

**Fix:**
```ruby
# Before:
[original code]

# After:
[corrected code]
```

**Verification:**
1. [Test step]
2. [Expected result]

---

## Summary

### Issues by Priority
| Priority | Count | Categories |
|----------|-------|------------|
| HIGH     | 2     | Security, Conventions |
| MEDIUM   | 3     | Performance, DRY |

### Critical Actions Required
1. Fix XSS vulnerability in user profile
2. Add missing authorization checks

### Positive Findings
✅ Excellent test coverage
✅ Proper use of transactions
✅ RESTful routing

### Overall Assessment
**Code Quality:** Good
**Security:** Needs attention (2 HIGH issues)
**Recommendation:** Fix security issues before merge
```

## Benefits of This System

### Consistency
- Same methodology across manual reviews, agent reviews, and command reviews
- Standardized output format
- Predictable quality

### Efficiency
- Slash command for instant review
- Agent for subagent dispatch
- Skill for manual review

### Maintainability
- Single source of truth (skill)
- Agent references skill
- Command references agent
- Update skill → everything improves

### Flexibility
- Use at appropriate level (command/agent/skill)
- Combine with other tools (`requesting-code-review`, `systematic-debugging`)
- Adapt to workflow needs

## Integration with Other Tools

### Works With
- `requesting-code-review` - Can dispatch this agent
- Git workflows - Reviews branches before merge
- PR processes - Pre-PR quality gate

### Complements
- Generic `code-reviewer` - This adds Rails-specific depth
- `systematic-debugging` - For investigating found issues
- `test-writer-fixer` - For adding test coverage

## Maintenance

### To Update Review Criteria
1. Edit `skills/rails-code-review/SKILL.md`
2. Add new checklist items, examples, or categories
3. Test changes
4. Agent and command automatically use updated criteria

### To Change Agent Behavior
1. Edit `agents/rails-code-reviewer.md`
2. Modify workflow steps or quality checklist
3. Command automatically uses updated agent

### To Customize Command
1. Edit `commands/rails-code-review.md`
2. Add project-specific instructions
3. Maintains reference to agent

## Success Metrics

A successful review session:
- ✅ Catches security vulnerabilities before merge
- ✅ Identifies Rails convention violations
- ✅ Suggests performance optimizations
- ✅ Provides actionable fixes with code examples
- ✅ Notes positive findings (not just problems)
- ✅ Gives clear merge/fix/refactor recommendation
- ✅ Takes 5-15 minutes depending on change size

## Proven Effectiveness

Tested on real Rails code (`public_deal_documents_without_requirement` branch):

**Issues Found:**
- 1 HIGH security issue (XSS vulnerability)
- 4 MEDIUM issues (authorization, strong params, performance)
- 3 LOW issues (duplication, clarity)
- 1 critical bug (mailer class name typo)

**Positive Findings Noted:**
- Excellent test coverage
- Proper use of transactions
- Correct async patterns
- RESTful routing

**Result:** Clear, actionable review with specific fixes and merge recommendation

## Getting Started

1. **Instant review:** `/rails-code-review`
2. **Manual review:** "Follow rails-code-reviewer agent"
3. **Learn technique:** Read `skills/rails-code-review/SKILL.md`

That's it! The system handles the rest.
