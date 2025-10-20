# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Alexandria is a modal GTD task management assistant operating via TaskWarrior MCP backend. This is a **prompt engineering project** - the "codebase" consists of 5 interlinked markdown documents defining LLM behavior through declarative specifications.

## System Architecture

**Runtime System (5-file constellation):**

1. **Alexandria_Prompt.md** - Main system prompt (1,121 tokens)
   - Identity, personality (confrontational honesty channeling Harlan Ellison)
   - Modal operation definitions (RECORD/SYNC/REVIEW/CLEANUP/REPORT)
   - Composite workflows (/morning, /eod, /weekly)
   - Startup protocol with DEGRADED mode detection
   - Command reference and critical rules

2. **Alexandria_MCP_Reference.md** - Tool assignments by mode (535 tokens)
   - Maps 16 TaskWarrior MCP tools to 5 operational modes
   - Essential parameters and field values for each tool
   - Tool usage boundaries enforcing modal purity
   - GTD tag structure (4 closed-set taxonomies)
   - Modal discipline protocols

3. **Alexandria_Inference_Rules.md** - Prolog-style inference specification (2,817 tokens)
   - Semantic reasoning protocol (ALL keyword lists are exemplars, applies to ALL predicates)
   - Rule precedence hierarchy (Priority 0-4)
   - Dependency chain parsing (7 vocabulary patterns: "then", "before", "after", etc.)
   - Ambiguity handling ("and" connector forces clarification - parallel vs sequential)
   - Project/context/status tag inference predicates
   - Quality flag scoring (🔴/⚠️) for vague input
   - Execution logic for RECORD mode only

4. **Alexandria_GTD_Reference.md** - Tag assessment frameworks (704 tokens)
   - Energy tag criteria (@focus/@admin/@creative/@review)
   - Time tag criteria (@quick/@medium/@long/@ongoing)
   - Status tag criteria (next/waiting/someday/active)
   - Context verification (@work/@personal)
   - Bulk retagging patterns for REVIEW mode
   - Cleanup red flags (age thresholds, modification frequency)
   - Age verification protocol (use get_task_info before staleness claims)

5. **Alexandria_Modal_Protocol.md** - State machine protocols (2,483 tokens)
   - DEGRADED mode specification (iPad primary use case - MCP unavailable)
   - Time check protocol (bash `date` always works)
   - MCP availability detection ("Tool not found" → DEGRADED)
   - Modal entry/exit protocols for all 5 modes
   - Write operation guards (SYNC before writes in connected mode)
   - Queue management (Q1/Q2/Q3 temporary IDs in DEGRADED)
   - Reconnection protocol with error handling
   - Silent quality flag integration (🔴/⚠️)

**Operational Flow:**
- Main prompt loads first, establishing identity and modes
- During RECORD: Applies inference rules from Alexandria_Inference_Rules.md
- During REVIEW: Uses GTD discipline frameworks for bulk tag assessment
- Before writes: Checks modal protocols for SYNC requirement
- In DEGRADED: Uses modal protocols for queue management (primary use case)

**Token Budget:** ~7,660 tokens total system definition

## Critical Constraints

**Closed-Set Taxonomies (NEVER INVENT NEW VALUES):**
- **Contexts:** @work, @personal
- **Status:** next, waiting, someday, active
- **Energy:** @focus, @admin, @creative, @review
- **Time:** @quick, @medium, @long, @ongoing
- **Quality Flags:** flag_red, flag_yellow

**Modal Discipline:**
- SYNC mandatory before ANY write operation in connected mode (task IDs are mutable)
- Never block capture - flag vague input (🔴/⚠️), clarify in REVIEW
- DEGRADED mode is primary use case (iPad without MCP backend)
- One mode, one purpose - no mixed operations
- Each mode uses ONLY assigned tools from MCP Tool Reference

**DEGRADED Mode (Primary Use Case):**
- MCP TaskWarrior backend unavailable (iPad, network issues)
- CAN: Check time via bash `date`, infer metadata, queue tasks locally (Q1, Q2, Q3...)
- CANNOT: Access TaskWarrior backend, execute SYNC/REVIEW/CLEANUP/REPORT
- Detection: Any MCP tool returns "Tool not found" error
- Reconnection: Sequential flush of queue with error handling

## Testing Strategy

**Holistic deployment testing** (not isolated component tests):

1. **Startup:** Load all 5 files, verify DEGRADED detection if MCP unavailable
2. **Inference - semantic matching:** "Debug authentication flow" → Alexandria_Development (semantic: "debug" matches "fix")
3. **Inference - vague input:** "Fix that thing" → 🔴, @personal, next (flag but don't block)
4. **Inference - dependency chain:** "Research hooks then write docs" → 2 tasks with dependency
5. **Inference - ambiguity:** "Research and write and test" → Asks clarification (sequential or parallel?)
6. **Modal discipline:** Execute `/morning` workflow, verify SYNC before operations
7. **DEGRADED simulation:** Capture tasks without MCP, verify queue (Q1, Q2...), test reconnection
8. **REVIEW workflow:** Execute `/weekly`, verify flagged tasks confronted first
9. **Closed-set enforcement:** Attempt to infer non-existent tag value, verify rejection

**Key validation points:**
- Semantic reasoning applied to all keyword-based inference (not just @work)
- Domain boundaries enforced ("book review" ≠ "code review")
- "and" connector triggers clarification, not failed parse
- No invented tag values (scope creep prevention)
- SYNC executed before writes in connected mode (no stale task ID errors)
- Flagged tasks (🔴/⚠️) confronted during REVIEW, not ignored
- DEGRADED mode continues RECORD without blocking
- Time checks use bash `date` first, not MCP
- Age claims verified via get_task_info before reporting staleness
- Clear tasks without project match → TaskWarrior inbox (backend handles, not inference)

## Design Philosophy

**Inference System:**
- **Semantic reasoning applies to ALL keyword-based inference** (projects, context, all predicates)
- Keywords are exemplars, not exhaustive - LLM uses natural language understanding
- Domain boundary rule: semantic similarity must be within same professional/topical domain
- Dependency chain parsing with 7 explicit vocabulary patterns ("then", "before", "after", etc.)
- "and" connector forces clarification (ambiguous: parallel vs sequential)
- Quality flags applied silently during capture, confronted in REVIEW
- Works in both connected and DEGRADED modes (inference-based, no MCP required)

**Modal Purity:**
- Each mode has specific purpose and assigned tools
- No cross-contamination (RECORD doesn't analyze, REVIEW doesn't capture)
- State guards prevent operations without fresh SYNC
- Composite workflows execute sequences with protocol discipline

**DEGRADED-First Design:**
- Majority of RECORD happens on iPad (DEGRADED mode)
- Queue management preserves full metadata for post-reconnection sync
- Graceful degradation with clear messaging
- bash `date` always available for time checking

**Confrontational Personality:**
- Channel Harlan Ellison: "That's busywork. Why does this matter?"
- Challenge sloppy thinking, force clarity
- No pleasantries, skip preambles
- Make thinking better, not easier

## File Modification Rules

**When editing prompts:**
- **Main prompt changes:** Alexandria_Prompt.md (identity, modes, workflows)
- **Tool assignments:** Alexandria_MCP_Reference.md (mode boundaries, parameters)
- **Inference logic:** Alexandria_Inference_Rules.md (Prolog predicates)
- **Tag assessment:** Alexandria_GTD_Reference.md (energy/time/status criteria)
- **Modal protocols:** Alexandria_Modal_Protocol.md (DEGRADED, SYNC, guards)

**When testing reveals issues:**
- Test holistically after changes (never isolated)
- Document in changelog with token impact analysis
- Create backup before major changes (ISO timestamp format)
- Token budget constraint: system definition should stay <10,000 tokens

## MCP Backend Dependency

Alexandria operates via TaskWarrior MCP server with two operational states:

**Connected mode (Desktop):** Full functionality
- All modes available (RECORD/SYNC/REVIEW/CLEANUP/REPORT)
- Write operations execute immediately after SYNC
- 16 MCP tools available (add_task, modify_task, get_next_tasks, etc.)

**DEGRADED mode (iPad - Primary Use Case):**
- MCP TaskWarrior backend unavailable
- RECORD continues with local queue (Q1, Q2, Q3... temporary IDs)
- Time check via bash `date` (always works)
- SYNC/REVIEW/CLEANUP/REPORT blocked with clear messaging
- Queue syncs on reconnection with error handling

**Detection:** Any MCP tool call returning "Tool not found" triggers DEGRADED mode

**Reconnection protocol:**
1. Detect MCP availability restored
2. Execute SYNC to get current backend state
3. Flush queue sequentially (add_task for each queued item)
4. Track successes/failures, report results
5. Clear queue (successful tasks only)
6. Final SYNC to refresh task IDs
7. Return to RECORD mode

## Development Workflow

**No traditional build/test commands** - this is prompt engineering, not software.

**Deployment testing:**
1. Create new conversation in Claude
2. Load all 5 markdown files in order (main prompt first)
3. Execute holistic test scenarios (see Testing Strategy above)
4. Validate against key validation points
5. Document token usage and behavior changes

**Iteration protocol:**
1. Identify issue via holistic testing
2. Determine which file(s) need updates
3. Create backup before major changes (ISO timestamp format)
4. Edit relevant file(s)
5. Re-test holistically (never assume isolated change works)
6. Measure token impact (production cost sensitivity)
7. Document changes with rationale

**Quality criteria:**
- **Internal consistency:** All tags use @ prefix, semantic reasoning scope explicit
- **Functional coherence:** Rules work together without conflicts or gaps
- No scope creep (closed-set taxonomies enforced)
- Modal discipline maintained (no tool boundary violations)
- DEGRADED mode continues functioning (primary use case)
- Token budget <10,000 total (currently ~7,800 after recent fixes)
- Confrontational personality preserved (no sycophantic drift)

## Prompt Engineering Principles

**From user's global CLAUDE.md:**
- Production prompts minimize markdown, use action tags like `<critical_notes>`
- No inline documentation or meta-commentary
- Info-level changelogs only
- Full paths only for file references
- Naming convention: Initial_Caps_With_Underscores.md

**Quality over verbosity:**
- "If the LLM needs this much hand-holding about 'you can reason,' the examples won't work anyway"
- Strip meta-commentary, trust semantic understanding
- Provide operational frameworks, not hand-holding
- Explicit constraints beat lengthy explanations

## Additional Files (Not Part of Runtime)

**Inference_System_Documentation.md** - Human-readable inference guide
- Explains architecture for developers, not loaded at runtime
- Contains example scenarios and semantic reasoning guidelines
- Reference for understanding Alexandria_Inference_Rules.md

**CLAUDE.md (this file)** - Development guide for Claude Code instances
- Not loaded at runtime with Alexandria
- Provides context for editing and testing the 5-file system
