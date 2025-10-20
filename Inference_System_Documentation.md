# Alexandria Inference System Documentation

## Overview

The Alexandria inference system automatically extracts metadata from natural language task descriptions during RECORD mode. It operates as a specification written in Prolog-style logic that guides LLM decision-making through closed-set taxonomies.

**Key principle**: Accept vague input without blocking capture, flag for later review.

## Architecture

### What It Does

When you dictate a task like "Fix the MCP bridge bug after standup with the team", the inference system:

1. **Detects structure** - Recognizes "after" indicates two tasks with dependency
2. **Assigns projects** - "MCP bridge" → Alexandria_Development, "standup" → work context
3. **Applies tags** - @work (team collaboration), next/waiting (dependency-based status)
4. **Creates dependencies** - "Fix bug" depends on "standup with team"
5. **Flags uncertainty** - No flags (description is specific, project matches)

### What It Doesn't Do

- **Never infers energy tags** (@focus/@admin/@creative/@review) - that's REVIEW mode's job
- **Never infers time tags** (@quick/@medium/@long/@ongoing) - also REVIEW mode
- **Never infers priority** (H/M/L) - explicit only
- **Never blocks capture** - accepts vague input with quality flags

## Rule Hierarchy (Priority Order)

### Priority 0: Task Structure Detection (HIGHEST)

Detects whether input describes one task or multiple tasks with dependencies.

**Trigger patterns:**
- Dependency vocabulary: "blocks", "depends on", "requires", "before", "after", "then", "once"
- Multiple action verbs in sequence

**Example:**
- "Research hooks then write docs" → 2 tasks, dependency chain
- "Fix the bug" → 1 task, no dependencies

### Priority 1: Explicit Project Mentions

Direct project references override all keyword matching.

**Patterns:**
- "Add to Alexandria backlog" → Alexandria_Development
- "For the LLM education project" → LLM_Education
- "for the centre" → Writing

### Priority 2: Multiple Keyword Resolution

When multiple keywords present, most specific rule wins.

**Example:**
- "Research TaskWarrior documentation" → Alexandria_Development (research beats writing)
- "Write TaskWarrior tutorial" → Writing (action verb "write" overrides topic)

### Priority 3: Development Keywords

**Keywords:** bug, fix, MCP, bridge, hooks, "research TaskWarrior"

**Assigned project:** Alexandria_Development

### Priority 4: Writing Keywords

**Keywords:** essay, article, blog (excluding TaskWarrior mentions)

**Assigned project:** Writing

### Default Fallback

If no rules match: apply @personal tag only, no project assignment.

## Dependency Chain Parsing

### Supported Vocabulary

**"then"** - Sequential tasks
- "Research hooks then write docs" → Task B depends on Task A

**"before"** - Reversed dependency
- "Write docs before publishing" → "Publishing" depends on "Write docs"

**"after"** - Standard dependency
- "Fix bug after meeting" → "Fix bug" depends on "Meeting"

**"depends on"** - Explicit dependency
- "Deploy depends on testing" → "Deploy" depends on "Testing"

**"blocks"** - Reversed dependency
- "Testing blocks deploy" → "Deploy" depends on "Testing"

**"requires"** - Standard dependency
- "Deploy requires approval" → "Deploy" depends on "Approval"

**"once"** - Conditional sequence
- "Once testing done, deploy" → "Deploy" depends on "Testing"

### Multi-task Output

When dependencies detected, system returns multiple task objects:

```
Input: "Research TaskWarrior hooks then write documentation"

Output:
- Task 1: "Research TaskWarrior hooks"
  - Project: Alexandria_Development
  - Tags: next, @personal

- Task 2: "Write documentation"
  - Project: Writing
  - Tags: waiting, @personal
  - Depends: "Research TaskWarrior hooks"
```

Note: Task 2 gets waiting status because it has dependencies.

## Status Tag Assignment

Status is represented as GTD-style tags (next/waiting/someday/active), NOT TaskWarrior's native status field.

**someday** - Explicit temporal marker
- Trigger: Contains phrase "Someday"
- Example: "Someday learn Rust" → someday

**waiting** - Blocked or delegated
- Trigger 1: Contains phrase "Waiting for"
- Trigger 2: Task has dependencies (automatic)
- Example: "Write docs" (in chain) → waiting

**next** - Ready to execute (DEFAULT)
- All tasks without blockers default to next

## Context Tag Assignment

Uses semantic reasoning - keywords are exemplars, not exhaustive lists.

**@work** - Team collaboration, professional deliverables
- **Exemplar keywords:** meeting, standup, sprint, code review, deployment, production, release, ticket, demo
- **Semantic matching:** "pull request review" matches even though "pull request" not listed (similar to "code review")

**@personal** - Individual projects, learning, life management (DEFAULT)
- Applied when no @work indicators present
- Example: "Research TaskWarrior hooks" → @personal

## Quality Flags

Automatically applied during capture to mark tasks needing clarification.

**flag_red (🔴)** - Vague description requiring confrontation
- Trigger 1: Contains pronouns only ("that thing", "the issue", "it", "this", "stuff")
- Trigger 2: No action verb present
- Example: "Fix that thing" → flag_red

**flag_yellow (⚠️)** - Low confidence classification
- Trigger: No project match found
- Example: "Buy groceries" → flag_yellow (no project, but clear description)

**REVIEW protocol:** Confront flagged tasks FIRST before other analysis.

## Ambiguity Handling

When structure detection finds multiple verbs but unclear dependency relationships:

1. **Evaluate sequence likelihood** (0.0 - 1.0 score based on verb count + connectors)
2. **If likelihood > 0.7** → Parse as dependency chain with inferred relationships
3. **If likelihood ≤ 0.7** → Request clarification, provide fallback (@personal, next)

**Connector handling:**
- **Explicit dependency:** "then", "before", "after", "depends on", "blocks", "requires", "once" → High confidence parsing
- **Ambiguous:** "and" → Forces likelihood = 0.5 (triggers clarification)
- **List separator:** "," (comma) → Contributes to likelihood but requires explicit connector

**Examples:**
- "Research hooks then write docs then test MCP" → Likelihood > 0.7, parse as chain
- "Research hooks and write docs and test MCP" → Likelihood = 0.5 (forced), ask: "Sequential or parallel?"
- "Research hooks, write docs, test MCP" → Likelihood calculated, may need clarification

## Exclusion Rules (RECORD Mode Boundaries)

These attributes are FORBIDDEN during capture - only applied manually in REVIEW mode:

**Energy tags:** @focus, @admin, @creative, @review

**Time tags:** @quick, @medium, @long, @ongoing

**Priority:** H, M, L (never inferred, always explicit)

**Annotations:** Never auto-generated

**Exception:** Dependencies ARE allowed when explicitly stated in input.

## Usage Pattern

### Single Entry Point

```prolog
execute_task_processing("RECORD", TaskDescription, Result)
```

- Mode is always "RECORD" for inference
- Other modes (SYNC/REVIEW/CLEANUP/REPORT) operate on existing tasks
- Returns inferred attributes following rule precedence

### Validation Requirements

1. **No pronouns in task descriptions** - "Fix it" rejected, "Fix MCP bridge" accepted
2. **Dependency chains require explicit task descriptions** - Can't reference tasks by pronouns
3. **Semantic reasoning applied** - Keyword lists are examples, not exhaustive

### Error Handling

**Vague input:** Accepted with flag_red, capture proceeds

**No project match:** Accepted with flag_yellow, @personal applied

**Ambiguous sequence:** Clarification requested, fallback provided

**Critical:** Never block capture. Flag and continue.

## Integration with Alexandria Modes

### RECORD Mode (Inference Active)
- Apply all inference rules
- Assign project, context, status tags
- Apply quality flags silently
- Parse dependencies automatically

### SYNC Mode (No Inference)
- Retrieve existing tasks only
- No metadata manipulation

### REVIEW Mode (No Inference)
- Operate on existing tasks using reporting tools
- Manually apply energy/time tags via bulk operations
- Confront flagged tasks
- No new task creation

### CLEANUP Mode (No Inference)
- Execute bulk modifications on existing tasks
- No new task creation

### REPORT Mode (No Inference)
- Generate summaries from existing tasks
- Read-only operations

## Semantic Reasoning Guidelines

**CRITICAL: Keyword lists in ALL predicates are EXEMPLARS, not exhaustive.**

This applies to: **project inference, context inference, all keyword-based matching.**

### How Semantic Matching Works

**Project Inference Examples:**
- Listed: "bug", "fix"
- Semantic matches: "debug", "troubleshoot", "patch", "resolve", "investigate"
- Listed: "essay", "article"
- Semantic matches: "post", "piece", "write-up", "tutorial", "guide"
- No match: "book review" for development keywords (different domain)

**Context Inference Examples:**
- Listed: "code review"
- Semantic matches: "pull request review", "design review", "PR review"
- Listed: "meeting", "standup"
- Semantic matches: "sync", "check-in", "1:1", "huddle"
- No match: "book review" (different domain)

### Domain Boundary Rule

Semantic similarity must be within the SAME professional/topical domain:
- ✅ "Debug" matches "fix" (both software development)
- ✅ "PR review" matches "code review" (both software collaboration)
- ❌ "Book review" does NOT match "code review" (different domains)
- ❌ "Essay" does NOT match "bug" (different domains)

Apply natural language understanding to recognize semantically similar concepts within the same professional domain.

## Example Inference Scenarios

### Scenario 1: Simple Task
**Input:** "Fix the MCP bridge bug"

**Inference:**
- Structure: Single task
- Project: Alexandria_Development (keyword "MCP")
- Tags: next (no blockers), @personal (no team indicators)
- Flags: None (clear description, project match)

### Scenario 2: Dependency Chain
**Input:** "Research TaskWarrior hooks then write documentation"

**Inference:**
- Structure: Dependency chain (keyword "then")
- Task 1: "Research TaskWarrior hooks"
  - Project: Alexandria_Development
  - Tags: next, @personal
- Task 2: "Write documentation"
  - Project: Writing (action verb "write")
  - Tags: waiting (has dependency), @personal
  - Depends: Task 1

### Scenario 3: Work Context
**Input:** "Fix bug after standup with team"

**Inference:**
- Structure: Dependency chain (keyword "after")
- Task 1: "Standup with team"
  - Project: Alexandria_Development (inferred from full context)
  - Tags: next, @work (keyword "standup")
- Task 2: "Fix bug"
  - Project: Alexandria_Development (keyword "bug")
  - Tags: waiting (has dependency), @work (inherited from context)
  - Depends: Task 1

### Scenario 4: Semantic Matching - Development
**Input:** "Debug the authentication flow"

**Inference:**
- Structure: Single task
- Project: Alexandria_Development (semantic: "debug" matches "fix", software development domain)
- Tags: next, @personal
- Flags: None (clear description, semantic project match)

### Scenario 5: Semantic Matching - Work Context
**Input:** "PR review for the API changes"

**Inference:**
- Structure: Single task
- Project: Alexandria_Development (semantic: "API changes" matches development domain)
- Tags: next, @work (semantic: "PR review" matches "code review")
- Flags: None (clear description)

### Scenario 6: Semantic Non-Match - Domain Boundary
**Input:** "Write book review for the team blog"

**Inference:**
- Structure: Single task
- Project: Writing (action verb "write" + "blog")
- Tags: next, @work (semantic: "team blog" indicates work collaboration)
- Flags: None
- Note: "book review" does NOT match development "review" (different domain)

### Scenario 7: Vague Input
**Input:** "Fix that thing"

**Inference:**
- Structure: Single task
- Project: None (no match)
- Tags: @personal (default), flag_red (vague: "that thing")
- Accepted for capture, flagged for REVIEW confrontation

### Scenario 8: No Project Match
**Input:** "Buy groceries"

**Inference:**
- Structure: Single task
- Project: None (no semantic match - grocery shopping is not in any project domain)
- Tags: @personal (default), flag_yellow (low confidence - no project)
- Accepted for capture, flagged for REVIEW classification

## Summary

The inference system provides **intelligent defaults during rapid capture** while maintaining **confrontational discipline during review**. It never blocks input, instead flagging uncertainty for later resolution.

**Semantic reasoning applies to ALL keyword-based inference** - project assignment, context tags, all predicates using keyword lists. Keywords are exemplars; the LLM uses natural language understanding to match semantically similar concepts within the same professional/topical domain.

All inference occurs in RECORD mode only - other modes operate on existing task data using reporting and bulk modification tools.
