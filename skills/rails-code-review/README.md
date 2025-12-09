# Rails Code Review Skill

Systematic Rails 8 code review skill with security-first approach, modern convention awareness, and pragmatic severity assessment.

## Quick Start

**Three ways to use:**

```bash
# 1. Via slash command (simplest)
/rails-code-review

# 2. Direct skill invocation
"Use the rails-code-review skill to review my changes"

# 3. Via requesting-code-review workflow
"Request code review for commits abc123..def456"
```

---

## Core Principles

This skill follows a **verify-first, pragmatic approach**:

1. ✅ **Verify before flagging** - If tests pass, verify claims before marking as issues
2. 🔒 **Security first** - Actual vulnerabilities over theoretical concerns
3. ⚡ **Pragmatic over pure** - Working non-standard code > Non-working standard code
4. 🎯 **Context matters** - Understands Rails 8 patterns and modern conventions

**Golden Rule:** If tests pass and code works, it's probably not broken.

---

## When to Use

### ✅ Use This Skill For:

- **Pre-merge reviews** - Catch issues before code reaches main
- **Pull request reviews** - Systematic PR evaluation
- **Security audits** - Focus on vulnerabilities
- **Convention checks** - Ensure Rails 8 best practices
- **Performance review** - Identify N+1 queries and bottlenecks

### ❌ Don't Use For:

- **Initial exploration** - Too early, code is still evolving
- **Non-Rails code** - Specific to Ruby on Rails
- **Infrastructure** - Use DevOps-specific skills
- **Trivial changes** - Typos, simple docs updates

---

## Workflow Integration

### Scenario 1: Quick Pre-Commit Check

**Use case:** You've made changes and want a quick sanity check before committing.

```bash
# Your workflow:
git add .
/rails-code-review  # Reviews uncommitted staged changes
# Review feedback
# Fix issues
git commit -m "feature: add drag and drop"
```

**Time:** 5-10 minutes
**Best for:** Small changes (< 200 lines)

---

### Scenario 2: Pre-PR Comprehensive Review

**Use case:** Feature branch ready, want thorough review before creating PR.

```bash
# Your workflow:
git checkout feature/drag-and-drop
git diff main...HEAD  # See what will be reviewed

# Request review
"Use the rails-code-review skill to review my feature branch against main"

# Skill will:
# 1. Get BASE_SHA (main) and HEAD_SHA (current)
# 2. Run bundle exec rspec to verify tests pass
# 3. Review git diff main...HEAD
# 4. Generate comprehensive report

# Review feedback, fix issues
# Create PR with confidence
```

**Time:** 15-30 minutes
**Best for:** Medium changes (200-500 lines)

---

### Scenario 3: Post-Implementation Agent Review

**Use case:** You've completed a task, want expert review before proceeding.

**Recommended:** Use `superpowers:requesting-code-review` workflow which dispatches this skill automatically.

```bash
# Your workflow:
git add . && git commit -m "implement feature"

# Request review with explicit commit range
BASE_SHA=$(git rev-parse HEAD~1)  # or origin/main
HEAD_SHA=$(git rev-parse HEAD)

# Use requesting-code-review skill (which calls this skill)
"Request code review for commits $BASE_SHA..$HEAD_SHA"
```

**Time:** 20-45 minutes
**Best for:** Any size change, most thorough review

---

### Scenario 4: Large PR Review (500+ lines)

**Use case:** Big feature, need structured review approach.

```bash
# Strategy: Review in phases

# Phase 1: Security & Migrations (critical)
"Use rails-code-review skill to check security issues in commits abc123..def456"

# Phase 2: Core logic & performance
"Use rails-code-review skill to check performance in app/models/ changes"

# Phase 3: Views & controllers
"Review controllers and views for Rails conventions"

# Phase 4: Full pass
/rails-code-review  # Final comprehensive check
```

**Time:** 60-120 minutes
**Best for:** Large changes (> 500 lines)

---

## Correct Usage Patterns

### Pattern 1: Explicit Commit Ranges (Recommended)

```bash
# ✅ GOOD: Explicit range
BASE_SHA=21b6733c
HEAD_SHA=d4a40ac1
"Review Rails changes in commits $BASE_SHA..$HEAD_SHA"
```

**Why:** Gives skill exact context, can verify diffs match tests.

### Pattern 2: Branch Comparison

```bash
# ✅ GOOD: Clear branch reference
"Review my feature/auth branch against main branch"
```

**Why:** Skill uses `git diff main...HEAD` to get changes.

### Pattern 3: Current Staged Changes

```bash
# ✅ GOOD: For pre-commit checks
git add app/controllers/users_controller.rb
"Review my staged changes"
```

**Why:** Quick feedback loop before commit.

### ❌ Anti-Patterns to Avoid

```bash
# ❌ BAD: No context
"Review my code"
# Skill doesn't know what to review

# ❌ BAD: Too vague
"Check if this is good"
# No specific focus area

# ❌ BAD: Review entire codebase
"Review all controllers for security"
# Should review changes, not entire codebase
```

---

## Integration with Other Skills

### With `superpowers:requesting-code-review`

**Best practice workflow:**

```bash
# 1. Complete your implementation
git add . && git commit -m "feature: implement drag-and-drop"

# 2. Request review (automatically uses rails-code-review)
"Request code review for this commit"

# 3. Requesting-code-review will:
#    - Get git diff
#    - Dispatch appropriate reviewer (rails-code-review for Rails)
#    - Generate comprehensive report
#    - Save to docs/code-reviews/

# 4. Review feedback, address issues

# 5. Request re-review if needed
"Request code review for latest commit"
```

### With `test-writer-fixer`

**Workflow for adding test coverage:**

```bash
# 1. Get code review
/rails-code-review
# Feedback: "Missing test coverage for Document#fulfill method"

# 2. Write tests
"Use test-writer-fixer to add tests for Document#fulfill"

# 3. Verify coverage
bundle exec rspec spec/models/document_spec.rb

# 4. Re-review
/rails-code-review
# Should now pass test coverage checks
```

### With `systematic-debugging`

**Workflow for fixing review issues:**

```bash
# 1. Get code review
/rails-code-review
# Feedback: "N+1 query detected in UsersController#index"

# 2. Debug the issue
"Use systematic-debugging to investigate N+1 query in UsersController#index"

# 3. Fix based on root cause

# 4. Verify fix
bundle exec rspec
/rails-code-review  # Should clear the issue
```

---

## Output Format

The skill generates **Markdown-formatted reports** with:

### Report Structure

```markdown
# Rails Code Review Report

**Branch:** `feature/drag-and-drop`
**Base:** `main`
**Commits:** `abc123..def456`
**Review Date:** 2025-12-09

## Executive Summary

[1-2 sentence overview]

**Overall Assessment:** [APPROVED / CONDITIONAL APPROVAL / BLOCK MERGE]

## Issues Found

### 🔴 Issue #1: Missing Authorization in Public Controller

**Category:** Security
**Priority:** CRITICAL
**Location:** `app/controllers/public/documents_controller.rb:10-17`

**Problem:**
[Description with code]

**Why It Matters:**
[Impact explanation]

**Fix:**
```ruby
# Before
def update
  document.update(params)
end

# After
def update
  authorize [:public, document], :update?
  document.update(params)
end
```

**Verification:**
```bash
bundle exec rspec spec/requests/public/documents_spec.rb
```

## Summary Table

| Priority | Category | Count | Issues |
|----------|----------|-------|--------|
| 🔴 CRITICAL | Security | 1 | Missing authorization |
| 🟡 HIGH | Performance | 2 | N+1 queries |
| 🟠 MEDIUM | Code Quality | 3 | Duplication |
| 🟢 LOW | CSS/Styling | 1 | Important flag |

## Positive Findings ✅

1. Excellent test coverage (98%)
2. Proper use of transactions
3. Clean Hotwire implementation

## Critical Actions Required

1. Add authorization checks (Issue #1)
2. Fix N+1 queries (Issues #2, #3)

## Overall Merge Recommendation

⚠️ **CONDITIONAL APPROVAL** - Fix critical issues first.
```

### Priority Levels

The skill uses **strict severity guidelines**:

| Priority | Criteria | Examples |
|----------|----------|----------|
| 🔴 **CRITICAL** | Security vulnerabilities, data loss, broken functionality | SQL injection, missing auth, 500 errors |
| 🟡 **HIGH** | Architecture problems, missing error handling, performance blockers | N+1 in main views, no rescue blocks |
| 🟠 **MEDIUM** | Non-standard patterns that work, duplication, missing tests | Works but unconventional, DRY violations |
| 🟢 **LOW** | Style, minor optimizations, documentation | CSS important flags, could extract constant |

**Key Rule:** If tests pass and code works, maximum priority is MEDIUM.

---

## Rails 8 Modern Patterns

The skill **understands and accepts** these modern Rails patterns:

### ✅ Will NOT Flag These

1. **Implicit Turbo Stream Responses**
   ```ruby
   def update
     @document.update(params)
     # No explicit render needed in Rails 8
   end
   ```

2. **where.missing Syntax** (Rails 7+)
   ```ruby
   scope :unassigned, -> { where.missing(:deal_requirement_document) }
   ```

3. **Concerns in Model Subdirectories**
   ```ruby
   # app/models/document/fulfillable.rb
   class Document
     module Fulfillable
       extend ActiveSupport::Concern
     end
   end
   ```

4. **broadcast_refresh_to**
   ```ruby
   broadcast_refresh_to(@deal)  # Valid full-page morph pattern
   ```

5. **Solid Stack Usage**
   - Solid Queue, Solid Cache, Solid Cable

---

## Pre-Review Validation

The skill performs **verification before flagging**:

```bash
# Skill automatically checks:
1. Are tests passing? (bundle exec rspec)
2. What's the Rails version? (Gemfile.lock)
3. What actually changed? (git diff BASE..HEAD)
4. Does the pattern work? (verification before flagging)
```

**This means:**
- Fewer false positives
- More accurate severity levels
- Context-aware recommendations
- Respects modern Rails conventions

---

## Common Mistakes & Solutions

### Mistake 1: Reviewing Without Commit Context

```bash
# ❌ Wrong
"Review my code for issues"

# ✅ Right
BASE=$(git rev-parse origin/main)
HEAD=$(git rev-parse HEAD)
"Review commits $BASE..$HEAD for security issues"
```

### Mistake 2: Expecting Instant Review of Large Changes

```bash
# ❌ Wrong expectation
git diff --stat  # Shows 1500 lines changed
/rails-code-review  # "Give me review in 2 minutes"

# ✅ Right approach
# Large PRs need 60+ minutes for thorough review
# Consider splitting PR or reviewing in phases
```

### Mistake 3: Ignoring Test Status

```bash
# ❌ Wrong
bundle exec rspec  # 5 failing tests
/rails-code-review  # Review anyway

# ✅ Right
bundle exec rspec  # Fix failing tests first
/rails-code-review  # Then review
```

### Mistake 4: Not Acting on Critical Issues

```bash
# ❌ Wrong
/rails-code-review
# Report: "🔴 CRITICAL: SQL injection vulnerability"
git push  # Push anyway

# ✅ Right
/rails-code-review
# Report: "🔴 CRITICAL: SQL injection vulnerability"
# Fix the issue immediately
bundle exec rspec  # Verify fix
/rails-code-review  # Re-review
git push  # Push when clean
```

---

## Troubleshooting

### Issue: "Skill flagged working code as broken"

**Solution:** The skill now verifies before flagging, but if this happens:

1. Check if tests actually pass: `bundle exec rspec`
2. Verify the pattern works in Rails 8
3. Provide feedback: the skill may need updating for new patterns

### Issue: "Review took too long"

**Expected times:**
- < 50 lines: 5-10 min
- 50-200 lines: 15-30 min
- 200-500 lines: 30-60 min
- 500+ lines: 60-120 min

**Solutions:**
- Split large PRs into smaller chunks
- Review in phases (security first, then logic, etc.)
- Use multiple reviewers for huge changes

### Issue: "Severity seems wrong"

**Check:**
- CRITICAL = Security vulnerability, data loss, or broken functionality
- HIGH = Architecture problem or performance blocker
- MEDIUM = Works but non-standard
- LOW = Style or optimization

If tests pass and code works, it can't be CRITICAL or HIGH.

### Issue: "False positive on Rails 8 pattern"

**Report it!** The skill should understand modern Rails patterns. If it incorrectly flags:
- `where.missing`
- Implicit Turbo responses
- `broadcast_refresh_to`
- Concerns in model subdirectories

This is a bug in the skill that should be fixed.

---

## Success Metrics

A successful review session should:

- ✅ Catch actual security vulnerabilities
- ✅ Identify real performance problems (N+1 queries)
- ✅ Suggest Rails 8 convention improvements
- ✅ Provide actionable fixes with code examples
- ✅ Note positive findings (not just problems)
- ✅ Give clear merge/fix recommendation
- ✅ Complete in reasonable time (< 30 min for typical PR)

---

## Advanced Usage

### Custom Focus Areas

```bash
# Focus on specific category
"Use rails-code-review skill to check security only"

# Focus on specific files
"Review security in app/controllers/public/documents_controller.rb"

# Focus on specific issue type
"Check for N+1 queries in the user listing changes"
```

### Incremental Review (Large PRs)

```bash
# Review phase by phase
"Review database migrations for safety"
# Fix issues

"Review models for performance"
# Fix issues

"Review controllers for security"
# Fix issues

"Review views for conventions"
# Final pass
```

### Re-Review After Fixes

```bash
# Initial review
/rails-code-review
# Output: "3 CRITICAL, 5 HIGH, 2 MEDIUM issues"

# Fix critical and high issues
# Commit fixes

# Re-review (only reviews new changes)
"Review my latest commit for the fixes"
# Output: "All CRITICAL and HIGH issues resolved"
```

---

## Integration with CI/CD

While this skill is designed for **human-driven reviews**, it can inform CI checks:

```yaml
# .github/workflows/review.yml (conceptual)
name: Rails Review
on: pull_request

jobs:
  automated_checks:
    runs-on: ubuntu-latest
    steps:
      # Run automated tools first
      - run: bundle exec brakeman -q  # Security
      - run: bundle exec rubocop      # Style
      - run: bundle exec rspec        # Tests

      # Then request human review with this skill
      # (Human reviews catch what tools miss)
```

**Note:** This skill is most effective as a **human-assisted review tool**, not a fully automated CI check.

---

## Maintenance & Updates

### Skill Location
**File:** `skills/rails-code-review/SKILL.md`

### Recent Improvements (December 2025)
- ✅ Added verify-first approach
- ✅ Rails 8 modern pattern awareness
- ✅ Stricter severity guidelines
- ✅ Proper Markdown output
- ✅ Pre-review validation steps
- ✅ Explicit git commit range handling

### How to Contribute Improvements

If you find patterns that should be:
- **Accepted:** Add to "Don't flag these" section
- **Flagged:** Add to appropriate priority checklist
- **Clarified:** Add examples to SKILL.md

---

## Quick Reference Card

```bash
┌─────────────────────────────────────────────────────┐
│         RAILS CODE REVIEW QUICK REFERENCE           │
├─────────────────────────────────────────────────────┤
│ QUICK START                                         │
│   /rails-code-review          # Instant review      │
│                                                     │
│ WITH COMMIT RANGE                                   │
│   BASE=$(git rev-parse origin/main)                 │
│   HEAD=$(git rev-parse HEAD)                        │
│   "Review commits $BASE..$HEAD"                     │
│                                                     │
│ REVIEW PRIORITIES                                   │
│   1. Security     (CRITICAL) - XSS, SQL injection   │
│   2. Conventions  (HIGH)     - Rails 8 Way          │
│   3. Performance  (HIGH)     - N+1, indexes         │
│   4. Duplication  (MEDIUM)   - DRY violations       │
│   5. Style        (LOW)      - CSS, formatting      │
│                                                     │
│ BEFORE REVIEW                                       │
│   bundle exec rspec           # Tests must pass     │
│                                                     │
│ AFTER CRITICAL ISSUES                               │
│   Fix immediately → Re-review → Then merge          │
│                                                     │
│ ESTIMATED TIMES                                     │
│   < 50 lines:   5-10 min                           │
│   50-200:       15-30 min                          │
│   200-500:      30-60 min                          │
│   500+:         60-120 min (consider splitting)    │
└─────────────────────────────────────────────────────┘
```

---

## Getting Help

**Skill not working as expected?**

1. Check Rails version: `grep "rails " Gemfile.lock`
2. Verify tests pass: `bundle exec rspec`
3. Provide explicit commit range
4. Check the skill file: `skills/rails-code-review/SKILL.md`

**Found a bug or false positive?**

Update the skill file or report to the skill maintainer.

**Need different review approach?**

Consider:
- `superpowers:code-reviewer` - General code review (non-Rails specific)
- `superpowers:systematic-debugging` - For investigating issues
- `superpowers:test-writer-fixer` - For test coverage gaps

---

## Summary

**rails-code-review** is a Rails 8-aware code review skill that:

✅ Understands modern Rails patterns
✅ Verifies before flagging issues
✅ Provides actionable fixes
✅ Outputs proper Markdown reports
✅ Integrates with development workflows
✅ Uses strict severity guidelines

**Best used for:** Pre-merge reviews, PR evaluation, security audits, convention checks, and performance optimization.

**Core philosophy:** Pragmatic, verification-first reviews that respect working code and understand Rails 8 conventions.
