---
name: rails-code-review
description: Use when reviewing Rails 8 code before merge or PR - systematic technique covering security vulnerabilities (XSS, SQL injection), Rails conventions, Hotwire/Turbo patterns, performance optimization, and DRY principles with priority-based issue categorization
---

# Rails Code Review Technique

## Overview

Systematic framework for reviewing Ruby on Rails 8 code focusing on security, conventions, performance, and maintainability. Prioritizes issues by impact (Critical → High → Medium → Low) and provides actionable fixes.

**Core principle:** Security first, conventions second, performance third, style last.

## When to Use

**Use this skill when:**
- Reviewing code before merging to main branch
- Creating or reviewing pull requests
- Performing pre-deployment code audits
- Reviewing changes that touch views, controllers, or models
- Investigating security vulnerabilities
- Optimizing Rails application performance

**Don't use this for:**
- Initial exploratory coding (too early)
- Non-Rails Ruby code
- Infrastructure/deployment configs (use other skills)
- Simple typo fixes or documentation updates

## Quick Reference

| Category | Focus Areas | Priority | Red Flags |
|----------|-------------|----------|-----------|
| **Security** | XSS, unsafe rendering, SQL injection, mass assignment | Critical | `html_safe`, `raw`, direct SQL, `params` in queries |
| **Rails Conventions** | Rails 8 Way, Solid Stack, Hotwire patterns | High | Custom JavaScript when Hotwire works, ignoring conventions |
| **Performance** | N+1 queries, caching, unnecessary Turbo | High | Missing `includes`, no caching, Turbo frame without need |
| **DRY** | Duplicate code, repeated patterns, extraction opportunities | Medium | Copy-pasted blocks, repeated partials, duplicate helpers |
| **CSS/Styling** | Tailwind v4, redundant classes, unused utilities | Low | Inline styles, conflicting classes, non-Tailwind CSS |
| **Browser Compat** | CSS properties, vendor prefixes | Low | Modern-only features, missing fallbacks |

## Implementation

### Step 1: Get the Diff

```bash
# Compare current branch against main
git diff main...HEAD
```

### Step 2: Category Scan (Priority Order)

Review changes in this exact order to catch critical issues first:

#### A. Security (CRITICAL PRIORITY)

**Check for:**
- [ ] XSS vulnerabilities in HAML/ERB rendering
- [ ] Unsafe string interpolation or `html_safe` usage
- [ ] `raw()` method calls without sanitization
- [ ] Direct SQL queries vulnerable to injection
- [ ] Mass assignment issues in controllers
- [ ] Unvalidated user input rendering
- [ ] Missing CSRF protection on forms

**Common vulnerabilities:**
```ruby
# ❌ BAD: XSS vulnerability
= raw(@user.bio)

# ✅ GOOD: Sanitized output
= sanitize(@user.bio, tags: %w[p br strong em])

# ❌ BAD: SQL injection
User.where("email = '#{params[:email]}'")

# ✅ GOOD: Parameterized query
User.where(email: params[:email])
```

#### B. Rails Conventions & Best Practices (HIGH PRIORITY)

**Check for:**
- [ ] Following "The Rails 8 Way" patterns
- [ ] Using Solid Stack over external dependencies
- [ ] Proper Hotwire/Turbo usage (not custom JavaScript)
- [ ] Turbo Frames have corresponding templates
- [ ] Controllers returning Turbo Streams have templates
- [ ] Using native Rails methods instead of custom solutions
- [ ] Proper use of Rails helpers and concerns
- [ ] RESTful routing and controller actions
- [ ] Following ActiveRecord conventions

**Anti-patterns:**
```ruby
# ❌ BAD: Custom JavaScript when Hotwire works
<%= link_to "Delete", user_path(@user),
    data: { confirm: "Are you sure?", method: :delete } %>
<script>/* custom delete handler */</script>

# ✅ GOOD: Turbo handles this
<%= link_to "Delete", user_path(@user),
    data: { turbo_method: :delete, turbo_confirm: "Are you sure?" } %>

# ❌ BAD: Controller returns turbo_stream without template
def create
  respond_to do |format|
    format.turbo_stream # No create.turbo_stream.erb file!
  end
end

# ✅ GOOD: Template exists or explicitly rendered
def create
  respond_to do |format|
    format.turbo_stream { render turbo_stream: turbo_stream.append(...) }
  end
end
```

#### C. Performance & Optimization (HIGH PRIORITY)

**Check for:**
- [ ] N+1 query problems (missing `includes`, `preload`, `eager_load`)
- [ ] Opportunities for caching or memoization
- [ ] Unnecessary database queries in loops
- [ ] Missing database indexes
- [ ] Inefficient ActiveRecord queries
- [ ] Turbo Frames/Streams used when not necessary
- [ ] Large payload responses

**Performance issues:**
```ruby
# ❌ BAD: N+1 query
@users.each do |user|
  user.posts.count  # Separate query for each user
end

# ✅ GOOD: Eager load
@users.includes(:posts).each do |user|
  user.posts.count  # Uses preloaded data
end

# ❌ BAD: Unnecessary complexity
respond_to do |format|
  format.turbo_stream  # For simple redirect
end

# ✅ GOOD: Simple redirect
redirect_to users_path, notice: "Created successfully"
```

#### D. CSS & Styling (MEDIUM PRIORITY)

**Check for:**
- [ ] Using Tailwind v4 syntax only
- [ ] No custom CSS (utility-first approach)
- [ ] Redundant or conflicting Tailwind classes
- [ ] Unused Tailwind utilities that can be removed
- [ ] Inline styles (should use Tailwind)
- [ ] Proper responsive breakpoints
- [ ] Consistent spacing and sizing patterns

**Styling issues:**
```haml
-# ❌ BAD: Conflicting classes
.flex.block.p-4.p-6

-# ✅ GOOD: Consistent classes
.flex.p-6

-# ❌ BAD: Inline styles
%div{ style: "margin-top: 20px;" }

-# ✅ GOOD: Tailwind utilities
.mt-5
```

#### E. Code Duplication (MEDIUM PRIORITY)

**Check for:**
- [ ] Repeated code blocks → extract to partials
- [ ] Repeated styling patterns → create shared component classes
- [ ] Duplicate logic → move to helpers, concerns, or decorators
- [ ] Similar views → use shared partials with locals
- [ ] Repeated queries → extract to scopes or query objects

**DRY opportunities:**
```haml
-# ❌ BAD: Repeated partial code
- @users.each do |user|
  .card.p-4.border.rounded
    .font-bold= user.name
    .text-gray-600= user.email

-# ✅ GOOD: Extract to partial
- @users.each do |user|
  = render 'user_card', user: user

-# _user_card.html.haml
.card.p-4.border.rounded
  .font-bold= user.name
  .text-gray-600= user.email
```

#### F. Bugs & Logic Issues (ALL PRIORITIES)

**Check for:**
- [ ] Logic errors or incorrect conditional logic
- [ ] Missing nil/null checks
- [ ] Missing error handling for external calls
- [ ] Race conditions in concurrent operations
- [ ] Incorrect data type handling
- [ ] Off-by-one errors in loops/ranges
- [ ] Missing edge case handling

**Note:** Specifically exclude:
- Accessibility issues (not a concern per project requirements)
- ARIA attributes (not a concern per project requirements)

#### G. Browser Compatibility (LOW PRIORITY)

**Check for:**
- [ ] CSS properties requiring vendor prefixes
- [ ] JavaScript features needing polyfills
- [ ] Browser-specific CSS quirks
- [ ] Features not supported in target browsers

### Step 3: Issue Documentation

For each issue found, provide:

1. **Category & Priority:** `[Security - CRITICAL]` or `[Performance - HIGH]`
2. **Location:** File path and line number(s)
3. **Problem:** Specific issue description
4. **Why it matters:** Security risk, performance impact, or maintenance concern
5. **Fix:** Concrete code example showing the correction
6. **Verification:** How to test the fix works

**Example output format:**

```
## [Security - CRITICAL] XSS Vulnerability in User Bio

**Location:** `app/views/users/show.html.haml:23`

**Problem:**
Using `raw()` to render user-submitted bio without sanitization.

**Why it matters:**
Users can inject malicious JavaScript that executes in other users' browsers,
potentially stealing session tokens or performing unauthorized actions.

**Fix:**
```haml
# Before:
= raw(@user.bio)

# After:
= sanitize(@user.bio, tags: %w[p br strong em a], attributes: %w[href])
```

**Verification:**
1. Create test user with bio: `<script>alert('xss')</script>`
2. View user profile
3. Should display escaped text, not execute script
```

## Common Mistakes

**When reviewing:**
- ❌ Reviewing line-by-line without understanding context
- ❌ Focusing on style issues before security/logic
- ❌ Missing Turbo Frame/Stream template mismatches
- ❌ Not checking for N+1 queries in list views
- ❌ Ignoring performance implications of eager loading vs lazy loading
- ❌ Accepting "it works" without checking security

**When providing feedback:**
- ❌ Vague criticism: "This could be better"
- ❌ Missing code examples in fixes
- ❌ Not explaining WHY something is an issue
- ❌ Mixing multiple priorities (critical + style) without distinction
- ❌ Not providing verification steps

## Integration

**Use with slash commands:**
```bash
/rails-code-review  # Triggers this skill automatically
```

**Use with requesting-code-review:**
After running automated code review, use this skill as a second pass for
Rails-specific patterns and conventions that generic reviewers might miss.

## Summary

**Review order:** Security → Conventions → Performance → DRY → Style → Browser

**For each issue:** Category + Priority + Location + Problem + Why + Fix + Verify

**Output:** Prioritized list with actionable fixes, not abstract suggestions.
