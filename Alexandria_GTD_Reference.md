# GTD Discipline for Alexandria REVIEW Mode

## Closed-Set Taxonomies

**NEVER INVENT NEW VALUES. Use only these tags:**

**Contexts:** @work, @personal
**Status:** next, waiting, someday, active
**Energy:** @focus, @admin, @creative, @review
**Time:** @quick, @medium, @long, @ongoing
**Quality Flags:** flag_red, flag_yellow

## Tag Application During RECORD Mode

Every task gets minimal tags during capture:
- **Context:** @work or @personal
- **Status:** next (default)

Energy and time tags applied during REVIEW only.

## REVIEW Mode Tag Assessment

### Energy Tags: Assess Cognitive Demand

**@focus:**
- Novel problem-solving requiring sustained concentration
- High decision density with multiple trade-offs
- Deep domain expertise required
- Examples: Architecture design, complex debugging, research synthesis

**@admin:**
- Routine operations with familiar patterns
- Low decision density, single clear action
- Brief attention blocks (<15 min)
- Examples: File management, scheduling, simple updates

**@creative:**
- Ideation and synthesis work
- Open-ended exploration with multiple valid approaches
- Generative rather than analytical
- Examples: Writing, brainstorming, design concepts

**@review:**
- Evaluation and comparison work
- Critical analysis of existing material
- Decision-making on completed work
- Examples: Code review, document assessment, prioritization

### Time Tags: Assess Task Scope

**@quick (<15 minutes):**
- Single atomic action
- No complex setup or validation
- Immediate completion
- No external dependencies

**@medium (15-60 minutes):**
- Multi-step process
- Some testing or validation required
- Minor dependencies
- Moderate complexity

**@long (>1 hour):**
- Complex deliverable with multiple components
- Significant testing and validation
- Multiple dependencies
- Substantial implementation work

**@ongoing (no completion):**
- Recurring activity or maintenance
- Continuous monitoring
- Process rather than deliverable
- No defined end state

### Status Tags: Assess Task Readiness

**next:**
- No blockers
- Information complete and ready to execute
- Default for capture

**waiting:**
- External dependency blocking progress
- Handed off to someone else
- Requires input before proceeding

**someday:**
- Future consideration, not time-committed
- Exploratory or aspirational
- Not ready to activate

**active:**
- Currently in progress
- User explicitly started work
- Has running timer

## Context Verification

**@work:**
- Professional responsibilities
- Team deliverables
- External stakeholders involved

**@personal:**
- Personal projects and maintenance
- Individual learning
- No external dependencies

## Bulk Retagging During REVIEW

Identify patterns:
- Tasks missing energy or time tags
- Tasks with inconsistent context
- Tasks with stale status

Apply bulk operations via modify_tasks_bulk with filters.

### Example Patterns

**Pattern 1: Project-based energy tags**
```
filter: "project:Alexandria_Development -@focus -@admin -@creative"
tags: ["@focus"]
```
Rationale: Development tasks typically require sustained concentration and deep domain expertise.

**Pattern 2: Description-based time tags**
```
filter: "description.contains:research -@quick -@medium -@long"
tags: ["@long"]
```
Rationale: Research tasks typically involve complex deliverables requiring >1 hour.

**Pattern 3: Dependency resolution**
```
filter: "description.contains:then -depends.any:"
```
Action: Review each task and manually add dependencies using modify_task.
Rationale: Tasks containing "then" likely describe sequences that should be linked.

**Pattern 4: Missing context tags**
```
filter: "-@work -@personal"
tags: ["@personal"]
```
Rationale: Default untagged tasks to personal context.

## Weekly Review Checklist

1. **Flagged tasks (🔴/⚠️):** Confront and clarify before proceeding
2. **Completed tasks:** Patterns in what got done vs ignored?
3. **Overdue tasks:** Wrong estimates or wrong priorities?
4. **Waiting tasks:** Still blocked or ready to unblock?
5. **Someday tasks:** Ready to activate?
6. **Tag accuracy:** Do energy/time tags match experience?

## Cleanup Red Flags

- Task age > 30 days with no progress
- Task modified > 5 times without completion
- Someday status > 90 days
- Conflicting tags (@work + @personal)
- No project assignment > 7 days

## Age Verification Protocol

Before applying age-based red flags, verify actual age via get_task_info:

**Fields to check:**
- `entry` - Task creation date
- `modified` - Last modification date

**Calculations:**
- Task age = current_time - entry
- Time since modification = current_time - modified

**Rule:** Uncertain about task age? Verify with get_task_info. Never guess.

## Tag Discipline

**Required during RECORD:** Context + status
**Required during REVIEW:** Energy + time (for tasks missing them)

Missing tags after REVIEW = needs classification.

## Quality Flag Implementation

**Storage Method**: TaskWarrior tags
- 🔴 Red flag → tag: `flag_red`
- ⚠️ Yellow flag → tag: `flag_yellow`

**Query Patterns**:
- Find flagged tasks: `+flag_red or +flag_yellow`
- Find red only: `+flag_red`
- Find yellow only: `+flag_yellow`

**Application**: Quality flags are automatically assigned during RECORD based on inference confidence:
- `flag_red`: Vague descriptions (pronouns only, no action verb)
- `flag_yellow`: Low confidence classification (no project match)

**Removal**: Remove flag after clarification via `tags_remove: ["flag_red"]` or `tags_remove: ["flag_yellow"]`

**REVIEW Protocol**: Confront flagged tasks before proceeding with other analysis
