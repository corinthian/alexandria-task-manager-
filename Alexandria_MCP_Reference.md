# Alexandria MCP Tool Reference

## Tool Assignments by Mode

### RECORD Mode (Default)
Rapid task capture and correction.

**Tools:**
- `add_task` - Create new task
- `modify_task` - Fix individual task
- `annotate_task` - Add timestamped note
- `append_task` - Add text to end
- `get_next_tasks` - Check existing tasks to find connections and avoid duplicates
- `prepend_task` - Add text to beginning
- `duplicate_task` - Copy existing task
- `undo_last` - Revert last operation

### SYNC Mode
State verification and retrieval.

**Tools:**
- `get_next_tasks` - List pending with urgency
- `list_tasks_filtered` - Query by criteria
- `get_task_info` - Full task details
- `count_tasks` - Numeric metrics

### REVIEW Mode
Tactical analysis and prioritization.

**Tools:**
- `builtin_report` - Reports: overdue, active, blocked, ready, completed
- `visualization_report` - Analytics: burndown, calendar, summary
- `count_tasks` - Status counts

### CLEANUP Mode
Task lifecycle management and bulk operations.

**Tools:**
- `mark_task_done` - Complete task
- `delete_task` - Remove permanently
- `modify_tasks_bulk` - Batch modifications
- `modify_task` - Individual updates
- `get_task_info` - Verify task age before staleness claims

### REPORT Mode
External summaries and stakeholder views.

**Tools:**
- `custom_report` - Filtered exports
- `visualization_report` - Charts/timesheets
- `builtin_report` - Formatted lists
- `get_task_info` - Verify task age for staleness reporting

## Essential Parameters

### Creating Tasks
```
add_task:
  description: "Task text"
  priority: H/M/L
  due: "2025-09-30"
  project: "ProjectName"
  tags: ["@context", "@energy", "@time", "@status"]
```

### Modifying Tasks
```
modify_task:
  identifier: "123" or UUID
  description: "Updated text"
  priority: H/M/L
  due: "2025-10-01"
  tags: ["new_tag"]
  tags_remove: ["old_tag"]
  clear_fields: ["priority", "due"]
  stop_task: true (deactivate running task)
```

### Bulk Operations
```
modify_tasks_bulk:
  filter: "project:Alexandria"
  priority: H
  tags: ["@work", "@focus"]
  tags_remove: ["old"]
  clear_fields: ["due"]
```

### Reports
```
builtin_report:
  report: "overdue"/"active"/"ready"/"completed"
  project: "ProjectName" (optional)
  priority: "H" (optional)

visualization_report:
  report: "burndown"/"calendar"/"summary"
  project: "ProjectName" (optional)

custom_report:
  report: "list"
  filter: "urgency.above:10 +@work"
```

## GTD Tag Structure

**Required per task:** 4 tags maximum
1. **Context:** @work or @personal
2. **Energy:** @focus, @admin, @creative, @review
3. **Time:** @quick (<15m), @medium (15-60m), @long (>1h), @ongoing
4. **Status:** next, waiting, someday, active

## Field Values

**Priority:** H (High), M (Medium), L (Low)
**Dates:** ISO format "YYYY-MM-DD"
**Status filters:** pending, completed, deleted, waiting, recurring
**Report types:** list, all, active, completed, blocked, overdue, ready, recurring

## Modal Discipline

Each mode uses ONLY its assigned tools. Complete current mode before switching.

**Mode entry:**
- RECORD: Default state
- SYNC: After changes or `/sync`
- REVIEW: Manual `/review`
- CLEANUP: Manual `/cleanup` or post-REVIEW
- REPORT: Manual `/report [type]`

**Composite workflows:**
- `/morning`: SYNC → REVIEW → RECORD
- `/eod`: RECORD → SYNC → Tomorrow preview
- `/weekly`: SYNC → REVIEW → CLEANUP → REPORT → SYNC
