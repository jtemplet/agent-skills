---
name: bd-issue-tracking
description: Use when work spans multiple sessions, has complex dependencies, or needs persistence across compaction - tracks issues with dependency graphs. TodoWrite for simple single-session tasks.
---

# bd Issue Tracking

## Overview

bd is a graph-based issue tracker for persistent memory across sessions. Use for multi-session work with complex dependencies; use TodoWrite for simple single-session tasks.

## Quick Start

**Every session start:**
1. `bd ready` - see available work
2. `bd list --status in_progress` - check active work
3. `bd show <issue-id>` - read notes for context
4. Report to user: "X items ready: [summary]"

**During work:**
- Update notes at milestones (70% token usage, blockers, major progress)
- Create issues for discovered work

**Critical:** Notes are your only memory after compaction. Write as if explaining to someone with zero conversation context.

## When to Use bd vs TodoWrite

### Use bd when:
- Multi-session work spanning multiple compaction cycles or days
- Complex dependencies with blockers or prerequisites
- Knowledge work (research, strategic documents, fuzzy boundaries)
- Side quests or exploratory work that might pause
- Need project memory to resume after weeks away

### Use TodoWrite when:
- Single-session tasks completing within current session
- Linear execution with no branching
- All information already in conversation
- Just need checklist to show progress

**Decision test:**
- "Will I need this context in 2 weeks?" Yes = bd
- "Could conversation history get compacted?" Yes = bd
- "Does this have blockers/dependencies?" Yes = bd
- "Will this be done in this session?" Yes = TodoWrite

**When in doubt: Use bd.** Better to have persistent memory you don't need than lose context you needed.

## Surviving Compaction

**Critical:** Compaction deletes conversation history but preserves beads.

**What survives:**
- All bead data (issues, notes, dependencies, status)
- Complete work history

**What doesn't survive:**
- Conversation history
- TodoWrite lists
- Recent discussion

**Notes format for post-compaction recovery:**
```
COMPLETED: Specific deliverables ("implemented JWT refresh endpoint + rate limiting")
IN PROGRESS: Current state + next step ("testing password reset, need user input on email template")
BLOCKERS: What's preventing progress
KEY DECISIONS: Important context or user guidance
NEXT: Immediate next action
```

**Notes quality check:**
- Could future-me resume with zero conversation history?
- Could another developer understand without asking?
- Are technical choices explained (not just stated)?
- Are user decisions captured?

**Good example:**
```
COMPLETED: JWT auth with RS256 (1hr access, 7d refresh tokens)
KEY DECISION: RS256 over HS256 per security review - enables key rotation
IN PROGRESS: Password reset flow - email service working, need rate limiting
BLOCKERS: Waiting on user: reset token expiry (15min vs 1hr trade-off)
NEXT: Implement rate limiting (5 attempts/15min) once expiry decided
```

## Progress Checkpointing

Update bd notes at these triggers:

**Critical triggers:**
- Token budget > 70% (proactively checkpoint)
- Token budget > 90% (automatically checkpoint)
- Major milestone reached
- Hit a blocker
- Task transition
- Before asking user for decision

**Checkpoint pattern:**
```bash
bd update <issue-id> --notes "COMPLETED: [what's done]
KEY DECISION: [important context]
IN PROGRESS: [current state]
BLOCKERS: [what's preventing progress]
NEXT: [immediate action]"
```

**Test:** "If compaction happened now, could future-me resume from these notes?"

## Session Start Protocol

bd auto-discovers database:
- Uses `.beads/*.db` in current project if exists
- Falls back to `~/.beads/default.db` otherwise

**Session start checklist:**
```
- [ ] Run bd ready to see available work
- [ ] Run bd list --status in_progress for active work
- [ ] If in_progress exists: bd show <issue-id> to read notes
- [ ] Report to user: "X items ready: [summary]"
- [ ] If nothing ready: bd blocked to check blockers
```

**Report pattern:** Establish shared context immediately without requiring user prompting.

## Core Operations

**Check ready work:**
```bash
bd ready
bd ready --priority 0           # Filter by priority
bd ready --assignee alice       # Filter by assignee
```

**Create issue (always quote titles):**
```bash
bd create "Fix login bug"
bd create "Add OAuth" -p 0 -t feature
bd create "Write tests" -d "Unit tests for auth module"
```

**Update status:**
```bash
bd update issue-123 --status in_progress
bd update issue-123 --notes "COMPLETED: [summary]. IN PROGRESS: [current]. NEXT: [action]"
```

**Close work:**
```bash
bd close issue-123
bd close issue-123 --reason "Implemented in PR #42"
```

**Show details:**
```bash
bd show issue-123               # Read notes for context
```

**List issues:**
```bash
bd list --status open
bd list --status in_progress
bd list --priority 0
```

**Check project health:**
```bash
bd stats                        # Total, open, closed, blocked, ready
bd blocked                      # Find blocked work
```

## Dependencies

**Four dependency types:**

1. **blocks** - Hard blocker (A blocks B from starting)
2. **related** - Soft link (related but not blocking)
3. **parent-child** - Hierarchical (epic/subtask)
4. **discovered-from** - Provenance (B discovered while working on A)

**Usage:**
```bash
bd dep add from-id to-id               # from blocks to
bd dep add auth-epic subtask --type parent-child
bd dep add main-task found-bug --type discovered-from
bd dep tree issue-123                  # Visualize dependencies
```

## Issue Lifecycle

**1. Discovery (proactive creation):**
```bash
# Found issue during work:
bd create "Found: auth doesn't handle profile permissions"
bd dep add current-task new-issue --type discovered-from
# Continue with original task
```

**2. Execution:**
```bash
bd update issue-123 --status in_progress
# Work on task, update notes at milestones
bd update issue-123 --notes "COMPLETED: [summary]..."
bd close issue-123 --reason "Implementation complete with tests"
```

**3. Planning (complex work):**
```bash
# Create parent epic
bd create "Implement user authentication" -t epic

# Create subtasks
bd create "Set up OAuth credentials" -t task
bd create "Implement authorization flow" -t task

# Link with dependencies
bd dep add auth-epic auth-setup --type parent-child
bd dep add auth-setup auth-flow    # blocks
```

## Integration with TodoWrite

**Temporal layering pattern:**

**TodoWrite (short-term - this hour):**
- Tactical execution: "Review Section 3", "Expand Q&A"
- Marked completed as you go
- Present/future tense
- Ephemeral: disappears at session end

**Beads (long-term - this week/month):**
- Strategic objectives: "Continue strategic planning document"
- Key decisions and outcomes in notes
- Past tense in notes
- Persistent: survives compaction

**Handoff pattern:**
1. Session start: Read bead → Create TodoWrite items
2. During work: Mark TodoWrite items completed
3. Reach milestone: Update bead notes with outcomes
4. Session end: TodoWrite disappears, bead survives

**Don't duplicate:** TodoWrite tracks execution, Beads captures meaning and context.

## Common Patterns

**Pattern 1: Knowledge Work Session**
```bash
# Session start
bd ready
# Returns: bd-42 "Research analytics expansion proposal" (in_progress)

bd show bd-42
# Notes show: "IN PROGRESS: Drafting cost-benefit analysis"

# Create TodoWrite for immediate work, mark completed as you go

# At milestone:
bd update bd-42 --notes "COMPLETED: Cost-benefit drafted.
KEY DECISION: User confirmed $50k budget - ruled out enterprise options.
IN PROGRESS: Finalizing recommendations.
NEXT: Get user review before closing."
```

**Pattern 2: Side Quest**
1. Discover problem during main task
2. Create issue: `bd create "Found: inventory needs refactoring"`
3. Link: `bd dep add main-task new-issue --type discovered-from`
4. Assess: blocker or defer?
5. If blocker: `bd update main-task --status blocked`, work new issue
6. If deferrable: note in issue, continue main task

**Pattern 3: Multi-Session Resume**
1. `bd ready` - available work
2. `bd blocked` - what's stuck
3. `bd list --status closed --limit 10` - recent completions
4. `bd show issue-id` - read context
5. Update status and begin

## Issue Creation Guidelines

**Quick checklist:**
- Title: Clear, specific, action-oriented
- Description: Problem statement (WHY) - immutable
- Design: HOW to build (can change)
- Acceptance: WHAT success looks like (stays stable)
- Priority: 0=critical, 1=high, 2=normal, 3=low
- Type: bug/feature/task/epic/chore

**Acceptance criteria test:**
"If I changed implementation, would these criteria still apply?"
- Yes = Good criteria (outcome-focused)
- No = Move to design field (implementation-focused)

**Example:**
- Good: "User tokens persist across sessions and refresh automatically"
- Wrong: "Use JWT tokens with 1-hour expiry" (that's design)

## Field Usage

| Field | Purpose | When to Set | Update Frequency |
|-------|---------|-------------|------------------|
| **description** | Immutable problem statement | At creation | Never |
| **design** | Initial approach, architecture | During planning | Rarely |
| **acceptance-criteria** | Deliverables checklist | When design clear | Mark completed |
| **notes** | Session handoff (COMPLETED/IN_PROGRESS/NEXT) | During work | At milestones |
| **status** | Workflow state (open/in_progress/closed) | As work progresses | When changing phases |
| **priority** | Urgency (0=highest, 3=lowest) | At creation | If priorities shift |

## Advanced Features

**JSON output (all commands):**
```bash
bd ready --json
bd show issue-123 --json
bd stats --json
```

**Bulk operations:**
```bash
bd close issue-1 issue-2 issue-3 --reason "Completed in sprint 5"
```

**Built-in help:**
```bash
bd quickstart                   # Comprehensive guide
bd create --help                # Command-specific help
```

## Troubleshooting

**Issues seem lost:** Use `bd list --status closed` (closed issues remain in database permanently)

**Dependencies wrong:** `bd show issue-id` shows full tree. Dependencies are directional: `bd dep add from-id to-id` means from blocks to.

**Database selection:** bd auto-discovers `.beads/*.db` in project or `~/.beads/default.db`. Use `--db /path/to/db` for explicit selection.

## Golden Rule

Write notes as if explaining to future-you with zero conversation context. After compaction, notes are your only memory.
