# Alexandria v1.0

**Modal GTD task management assistant powered by LLM inference**

[![Version](https://img.shields.io/badge/version-1.0-blue.svg)](https://github.com/yourusername/alexandria)

## Prerequisites

- **TaskWarrior MCP server** installed and configured ([mcp-server-taskwarrior](https://github.com/corinthian/mcp-server-taskwarrior))
- **Claude Sonnet 4+** access (semantic reasoning works best with Sonnet 4+)

## Overview

Alexandria is a **prompt engineering project**, not traditional software. It's a 5-file LLM system that converts natural language task descriptions into structured GTD tasks through semantic inference and modal operation.

**What it does:**
- Converts "Fix MCP bridge bug after standup" → 2 tasks with dependency, correct projects, GTD tags
- Operates through 5 distinct modes (RECORD/SYNC/REVIEW/CLEANUP/REPORT)
- Works offline (DEGRADED mode) with local queue, syncs when reconnected
- Applies confrontational honesty: challenges vague thinking, forces clarity

**What makes it different:**
- DEGRADED-first design (works without backend connectivity)
- Semantic reasoning (keywords are exemplars, not exhaustive)
- Never blocks capture (flags vague input, clarifies later)
- Modal discipline (one mode, one purpose, strict tool boundaries)

**Token budget:** ~7,800 tokens total system definition (production cost constraint)

## Architecture

Alexandria consists of 5 markdown files (~7,800 tokens total) loaded into a Claude conversation. The system auto-configures on startup: detects MCP backend availability, loads inference rules, and enters RECORD mode ready to capture tasks.

## External Dependencies

### TaskWarrior MCP Server (REQUIRED for connected mode)

Alexandria operates via [TaskWarrior MCP server](https://github.com/corinthian/mcp-server-taskwarrior), which provides 16 task management tools through the Model Context Protocol standard.

**Two operational states:**

**Connected mode (Desktop):** Full functionality
- All modes available (RECORD/SYNC/REVIEW/CLEANUP/REPORT)
- Write operations execute immediately after SYNC
- 16 MCP tools: `add_task`, `modify_task`, `get_next_tasks`, `mark_task_done`, etc.

**DEGRADED mode (iPad/Mobile - PRIMARY USE CASE):** Graceful degradation
- MCP TaskWarrior backend unavailable
- RECORD continues with local queue (Q1, Q2, Q3... temporary IDs)
- Time check via `bash date` (always works)
- SYNC/REVIEW/CLEANUP/REPORT blocked with clear messaging
- Queue syncs on reconnection with error handling

**Detection:** Any MCP tool call returning "Tool not found" → DEGRADED mode

## Usage

### Deployment
1. Create new conversation in Claude (Sonnet 4+ recommended)
2. Load 5 runtime markdown files in order:
   - Alexandria_Prompt.md (loads first)
   - Alexandria_MCP_Reference.md
   - Alexandria_Inference_Rules.md
   - Alexandria_GTD_Reference.md
   - Alexandria_Modal_Protocol.md
3. System auto-detects MCP availability
4. Enters RECORD mode (ready to capture tasks)

### When to Use Which Mode

**Just capturing tasks?** Stay in RECORD mode (automatic) - just dictate tasks as they come.

**Need to organize/prioritize?** Run `/review` to apply energy/time tags and confront flagged tasks.

**Things piling up?** Run `/weekly` (Fridays/weekends) for full GTD discipline: review, cleanup, planning.

**Daily workflows:** Use `/morning` (start of day) and `/eod` (end of day) for quick check-ins.

### Commands
- `/sync` - Force synchronization with TaskWarrior
- `/review` - Enter REVIEW mode (analyze and tag)
- `/cleanup` - Enter CLEANUP mode (prune stale tasks)
- `/report [type]` - Generate stakeholder report
- `/morning` - Morning workflow (SYNC → brief REVIEW → RECORD)
- `/eod` - End of day workflow (final capture → SYNC → preview tomorrow)
- `/weekly` - Weekly review (full GTD discipline, Fridays/weekends)
- `/status` - Current mode and queue metrics

### Example Session

```
User: "Fix MCP bridge bug after standup with team"

Alex: [RECORD] 2 tasks captured:
      Task 47: Standup with team → Alexandria_Development @work next
      Task 48: Fix MCP bridge bug → Alexandria_Development @work waiting
               (depends: Task 47)

User: "That thing from yesterday"

Alex: [RECORD🔴] Task 49 → @personal next flag_red
      (Vague - will confront during REVIEW)

User: /weekly

Alex: [SYNC] 47 tasks retrieved
      [REVIEW] 1 flagged task requires clarification:

      Task 49: "That thing from yesterday"

      What did you mean? Be specific.
```

## Design Philosophy

### DEGRADED-First Design
- Majority of RECORD happens on iPad (no MCP backend)
- Queue management preserves full metadata for post-reconnection sync
- Graceful degradation with clear messaging
- `bash date` always available for time checking

### Never Block Capture
- Accept vague input with quality flags
- Clarify during REVIEW, not at capture
- Inference-based flagging (works in both connected and DEGRADED modes)

### Confrontational Personality
Channel Harlan Ellison's confrontational honesty:
- "That's busywork. Why does this matter?"
- "This task has been 'waiting' for 47 days. It's dead."
- Challenge sloppy thinking, force clarity
- Make thinking better, not easier

## Customization Guide

**Can I use this without customization?** Yes - the system works out-of-box with default projects (Alexandria_Development, Writing, Taskwarrior_integration). Tasks that don't match existing projects default to @personal with flag_yellow (low confidence). You'll want to add your own projects to reduce flagged tasks.

### Quick Start: Essential Personalizations

Every user should customize these 4 items:

1. **User name** (Alexandria_Prompt.md:120)
   - Find: "You're not here to make Richard feel productive"
   - Replace: "Richard" with your name

2. **Projects** (Alexandria_Inference_Rules.md:27-64)
   - Add your own project inference predicates
   - See "Pattern 1: Add New Project" below

3. **Personality calibration** (Alexandria_Prompt.md:70-76)
   - Adjust confrontational examples to match your style
   - Or keep Harlan Ellison intensity as-is

4. **Staleness thresholds** (Alexandria_GTD_Reference.md:159-163)
   - Default: 30 days age, 90 days someday, 5 modifications
   - Adjust based on your workflow cadence

### Customization Complexity Matrix

| What to Change | File(s) | Complexity | Breaks If Wrong |
|----------------|---------|------------|-----------------|
| Projects | Inference_Rules.md | **Low** | Nothing (falls back to @personal) |
| Personality | Prompt.md | **Low** | Nothing |
| Energy criteria | GTD_Reference.md | **Low** | Nothing (just assessment accuracy) |
| Staleness thresholds | GTD_Reference.md | **Low** | Nothing (just cleanup triggers) |
| Dependency vocabulary | Inference_Rules.md | **Medium** | Chain parsing fails |
| Work/personal keywords | Inference_Rules.md | **Medium** | Context mis-tagging |
| Closed-set taxonomy values | 3+ files | **High** | Modal discipline, inference, validation |
| Modal tool assignments | MCP_Reference.md | **High** | Mode operations fail |
| SYNC protocol | Modal_Protocol.md | **Critical** | Data corruption (stale IDs) |

### Common Customization Patterns

#### Pattern 1: Add New Project

**Use case:** You have a project domain not covered by default inference

1. Open `Alexandria_Inference_Rules.md`, find `% Priority 3: Development Keywords`
2. Add new predicate with your keywords:
   ```prolog
   infer_project(Task, Project) :-
       contains_any_keyword(Task, ["workout", "exercise", "gym"]),
       Project = "Fitness_Routine".
   ```
3. Test: "Schedule workout" → should infer Fitness_Routine project

#### Pattern 2: Customize Energy Tag Criteria

**Use case:** Default cognitive demand categories don't match your work style

1. Open `Alexandria_GTD_Reference.md`, find `### Energy Tags: Assess Cognitive Demand`
2. Modify criteria descriptions to match your work patterns
3. Example: Change `@focus` from "novel problem-solving" to "deep work requiring >30min uninterrupted"

#### Pattern 3: Add Dependency Keyword

**Use case:** You use different vocabulary for task sequences

1. Open `Alexandria_Inference_Rules.md`, find `### Task Dependency Parsing Rules`
2. Add new pattern matching your vocabulary (e.g., "following", "preceding", "contingent on")
3. Test: "Complete review following draft" → should create 2 tasks with dependency

#### Pattern 4: Change Personality Style

**Use case:** Confrontational honesty too aggressive/not aggressive enough

1. Open `Alexandria_Prompt.md`, find `## PERSONALITY CALIBRATION`
2. Replace example phrases with your preferred tone (softer: "Still relevant?", harder: "Delete it or ship it")
3. Update user name reference to yours

### Advanced: Changing Closed-Set Taxonomies

**⚠️ WARNING: High complexity, requires coordinated updates across 3-5 files**

Changing core taxonomies (contexts, status, energy, time tags) requires updating definitions in `MCP_Reference.md`, inference predicates in `Inference_Rules.md`, and assessment criteria in `GTD_Reference.md`. After changes, grep all files for old values and test holistically. See Complexity Matrix above for risk assessment.

### Modification Safety Checklist

**Before editing any file:**
- ✅ Create git commit or timestamped backup
- ✅ Note current token count for changed file
- ✅ Identify which modes use this file (see File Reference Guide)
- ✅ Check for cross-file dependencies: `grep -r "key_term" *.md`

**After editing:**
- ✅ Measure new token count (stay under 10k total budget)
- ✅ Test holistically (load all 5 files, run affected scenarios)
- ✅ Validate against quality criteria (consistency, coherence, no scope creep)
- ✅ Update CLAUDE.md if architecture changes

### Platform-Specific Considerations

**If you don't use iPad:**
- DEGRADED mode less relevant for you
- Can remove iPad references from documentation
- Keep DEGRADED functionality (works for any MCP unavailability: network failures, backend crashes)

**If always connected to MCP:**
- Can simplify queue management in Modal_Protocol.md
- Still keep DEGRADED detection (failures happen)
- Consider reducing DEGRADED protocol detail (lines 365-485 in Modal_Protocol.md)

**If using different task backend (not TaskWarrior):**
- **Alexandria_MCP_Reference.md needs complete rewrite**
- Map tool assignments to your backend's API
- Closed-set taxonomies must match backend schema
- SYNC protocol must account for backend's ID stability guarantees
- Example: Todoist, TickTick, Org-mode have different field structures

## Troubleshooting Common Customization Issues

**New project not inferring:** Check if predicate added to Inference_Rules.md, verify keywords are semantically related, confirm no higher-priority rule overriding yours.

**Inference behaving unpredictably:** Review rule precedence hierarchy (Priority 0-4), check for conflicting predicates at same priority level, test with single task to isolate issue.

**Modal discipline breaking (stale task ID errors):** Ensure SYNC runs before all write operations, verify you didn't change tool assignments without updating protocols.

## Examples Gallery

### Example 1: Marketing Professional Customization

**Projects:**
```prolog
infer_project(Task, Project) :-
    contains_any_keyword(Task, ["campaign", "ad", "launch", "promotion"]),
    Project = "Campaign_Execution".

infer_project(Task, Project) :-
    contains_any_keyword(Task, ["brand", "positioning", "messaging"]),
    Project = "Brand_Strategy".

infer_project(Task, Project) :-
    contains_any_keyword(Task, ["content", "post", "blog", "social"]),
    Project = "Content_Calendar".
```

**Energy tags (customized):**
```markdown
**@strategic:**
- High-level planning requiring creative thinking
- Campaign strategy, brand positioning
- Best in morning when fresh

**@execution:**
- Implementation tasks, asset creation
- Ad setup, content publishing
- Can do with moderate energy

**@review:**
- Performance analysis, metrics review
- A/B test evaluation, reporting
- Analytical work requiring focus
```

**Personality:** Collaborative instead of confrontational
```markdown
Channel supportive accountability:
- "This campaign has been in planning for 3 weeks. Ready to launch?"
- "Great progress on content calendar. What's blocking the next step?"
```

### Example 2: Academic Researcher Customization

**Projects:**
```prolog
infer_project(Task, Project) :-
    contains_any_keyword(Task, ["write", "draft", "paper", "manuscript"]),
    Project = "Paper_Writing".

infer_project(Task, Project) :-
    contains_any_keyword(Task, ["read", "review", "survey", "literature"]),
    Project = "Literature_Review".

infer_project(Task, Project) :-
    contains_any_keyword(Task, ["analyze", "data", "statistics", "model"]),
    Project = "Data_Analysis".

infer_project(Task, Project) :-
    contains_any_keyword(Task, ["grant", "proposal", "funding"]),
    Project = "Grant_Applications".
```

**Time tags (customized for research sessions):**
```markdown
**@session:**
- Full research session (2-4 hours)
- Writing, deep analysis, literature review
- Requires uninterrupted time block

**@task:**
- Single discrete task (<1 hour)
- Data cleaning, figure generation, email
- Can fit between meetings
```

**Context (customized):**
```markdown
**Contexts:** @lab, @office, @remote

**@lab:**
- Requires physical lab access
- Experiments, equipment, wet bench work

**@office:**
- On-campus but not lab-specific
- Meetings, collaboration, library access

**@remote:**
- Can do from anywhere
- Writing, analysis, literature review
```

### Example 3: Freelancer Customization

**Projects (client-based):**
```prolog
infer_project(Task, Project) :-
    contains_any_keyword(Task, ["ClientA", "Acme", "Acme Corp"]),
    Project = "Client_Acme".

infer_project(Task, Project) :-
    contains_any_keyword(Task, ["ClientB", "TechCo"]),
    Project = "Client_TechCo".

infer_project(Task, Project) :-
    contains_any_keyword(Task, ["proposal", "pitch", "lead", "prospect"]),
    Project = "Business_Development".

infer_project(Task, Project) :-
    contains_any_keyword(Task, ["invoice", "accounting", "tax", "admin"]),
    Project = "Business_Admin".
```

**Status tags (customized for billing):**
```markdown
**Status:** next, waiting, someday, active, invoiced

**invoiced:**
- Work completed and billed
- Waiting for payment
- Track via tag for billing reminders
```

**Energy (billing-focused instead of cognitive):**
```markdown
**Energy:** @billable, @non_billable, @development, @admin

**@billable:**
- Direct client work
- Tracked hours, invoiceable
- Prioritize during working hours

**@non_billable:**
- Business development, marketing
- Important but not directly revenue-generating
- Schedule in off-peak times
```

**Personality (client-aware):**
```markdown
Channel professional urgency:
- "Client deliverable due Friday. On track?"
- "Invoice sent 30 days ago. Follow up on payment?"
- "This proposal has been drafted for 2 weeks. Time to send or archive?"
```

---

These examples demonstrate different customization philosophies:
- **Marketing:** Collaborative personality, strategic/execution energy split
- **Academic:** Session-based time tags, location contexts (lab/office/remote)
- **Freelancer:** Client-based projects, billing-aware energy tags, status tracking

The system is flexible enough to adapt to different professional contexts while maintaining core GTD discipline and modal architecture.

## Contributing

For development guidelines, testing protocols, and quality standards, see CLAUDE.md.

## License

[![CC BY 4.0][cc-by-shield]][cc-by]

This work is licensed under a [Creative Commons Attribution 4.0 International License][cc-by].

Some rights reserved - Attribution only.

[cc-by]: http://creativecommons.org/licenses/by/4.0/
[cc-by-shield]: https://img.shields.io/badge/License-CC%20BY%204.0-lightgrey.svg

## Acknowledgments

- GTD methodology by David Allen
- TaskWarrior MCP server by Model Context Protocol community
- Inspired by Harlan Ellison's confrontational honesty
