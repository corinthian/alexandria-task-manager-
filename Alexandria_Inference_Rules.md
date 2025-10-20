## Core Inference Engine Configuration

### Semantic Reasoning Protocol

**CRITICAL: All keyword lists in ALL predicates are EXEMPLARS, not exhaustive.**

Apply natural language understanding to recognize semantically similar concepts within the same domain:
- `contains_any_keyword(Task, ["bug", "fix"])` matches "debug", "troubleshoot", "patch", "resolve"
- `contains_any_keyword(Task, ["meeting", "standup"])` matches "sync", "check-in", "1:1", "huddle"
- Semantic similarity must be within the SAME professional/topical domain
- "Book review" does NOT match "code review" (different domains)

This applies to: project inference, context inference, all keyword-based predicates.

### Rule Precedence Hierarchy (Highest to Lowest Priority)

```prolog
% Priority 0: Task Dependency Structure Detection (HIGHEST)
infer_task_structure(Input, StructureType) :-
    contains_dependency_vocabulary(Input, ["blocks", "depends on", "requires", "before", "after", "then", "once"]),
    StructureType = "dependency_chain".

infer_task_structure(Input, StructureType) :-
    multiple_verbs_in_sequence(Input),
    StructureType = "potential_chain".

% Priority 1: Explicit Project Mentions
infer_project(Task, Project) :-
    contains_phrase(Task, "Add to Charlie backlog"),
    Project = "Charlie_research".

infer_project(Task, Project) :-
    contains_phrase(Task, "For the Alpha project"),
    Project = "Alpha_analysis".

infer_project(Task, Project) :-
    contains_phrase(Task, "the blog"),
    Project = "Writing".

% Priority 2: Multiple Keyword Resolution (Most Specific Wins)
infer_project(Task, Project) :-
    contains_keywords(Task, ["Draft", "documentation"]),
    \+ action_verb_present(Task, ["write", "draft", "compose"]),
    Project = "Bravo_blog".

infer_project(Task, Project) :-
    action_verb_present(Task, ["write", "draft", "compose"]),
    contains_keyword(Task, "edit", "correct", "revise"),
    Project = "Writing".

% Priority 3: Development Keywords (SEMANTIC MATCHING - see Semantic Reasoning Protocol)
% Exemplars: bug, fix, MCP, bridge, hooks, research TaskWarrior
% Semantic matches: debug, troubleshoot, patch, resolve, investigate, refactor
infer_project(Task, Project) :-
    contains_any_keyword(Task, ["bug", "fix", "MCP", "bridge", "hooks", "research TaskWarrior"]),
    Project = "Taskwarrior_integration".

% Priority 4: Writing Keywords (SEMANTIC MATCHING - see Semantic Reasoning Protocol)
% Exemplars: essay, article, blog
% Semantic matches: post, piece, write-up, tutorial, guide
infer_project(Task, Project) :-
    contains_any_keyword(Task, ["essay", "article", "blog"]),
    \+ contains_keyword(Task, "TaskWarrior"),
    Project = "Writing".
```

### Task Dependency Parsing Rules

```prolog
% Dependency Vocabulary Detection
contains_dependency_vocabulary(Input, Vocab) :-
    member(Word, Vocab),
    contains_phrase(Input, Word).

% Sequential Action Detection
multiple_verbs_in_sequence(Input) :-
    findall(Verb, (action_verb(Verb), contains_word(Input, Verb)), Verbs),
    length(Verbs, Count),
    Count > 1.

% Dependency Chain Parsing
parse_dependency_chain(Input, TaskChain) :-
    contains_phrase(Input, " then "),
    split_on_phrase(Input, " then ", Tasks),
    create_dependency_chain(Tasks, TaskChain).

parse_dependency_chain(Input, TaskChain) :-
    contains_phrase(Input, " before "),
    split_on_phrase(Input, " before ", [TaskA, TaskB]),
    TaskChain = [dependency(TaskB, TaskA)].

parse_dependency_chain(Input, TaskChain) :-
    contains_phrase(Input, " after "),
    split_on_phrase(Input, " after ", [TaskA, TaskB]),
    TaskChain = [dependency(TaskA, TaskB)].

parse_dependency_chain(Input, TaskChain) :-
    contains_phrase(Input, "depends on"),
    split_on_phrase(Input, "depends on", [TaskA, TaskB]),
    TaskChain = [dependency(TaskA, TaskB)].

parse_dependency_chain(Input, TaskChain) :-
    contains_phrase(Input, "blocks"),
    split_on_phrase(Input, "blocks", [TaskA, TaskB]),
    TaskChain = [dependency(TaskB, TaskA)].

parse_dependency_chain(Input, TaskChain) :-
    contains_phrase(Input, "requires"),
    split_on_phrase(Input, "requires", [TaskA, TaskB]),
    TaskChain = [dependency(TaskA, TaskB)].

parse_dependency_chain(Input, TaskChain) :-
    contains_phrase(Input, "once"),
    extract_conditional_sequence(Input, TaskChain).

% Create Sequential Dependencies
create_dependency_chain([Task], [task(Task)]).
create_dependency_chain([First|Rest], [task(First), dependency(Second, First)|ChainRest]) :-
    Rest = [Second|_],
    create_dependency_chain(Rest, [task(Second)|ChainRest]).
```

### GTD Status Tag Inference Rules

```prolog
% GTD Status as Tags (not TaskWarrior native status)
% TaskWarrior native status is always "pending" for active tasks

% Temporal Status Tag Overrides
infer_status_tag(Task, "someday") :-
    contains_phrase(Task, "Someday").

infer_status_tag(Task, "waiting") :-
    contains_phrase(Task, "Waiting for").

% Dependency-based Status Tag
infer_status_tag(Task, "waiting") :-
    has_dependencies(Task).

% Default Status Tag
infer_status_tag(_, "next").
```

### Tag Inference Rules

```prolog
% Work Context: Team collaboration, professional deliverables, organizational processes (SEMANTIC MATCHING)
% Exemplars: meeting, standup, sprint, code review, deployment, production, release, ticket, demo
% Semantic matches: sync, check-in, 1:1, huddle, PR review, ship, launch, push, issue
infer_tag(Task, "@work") :-
    contains_any_keyword(Task, ["meeting", "standup", "sprint", "code review", "deployment", "production", "release", "ticket", "demo"]).

% Personal Context: Individual projects, learning, personal life management (DEFAULT)
infer_tag(Task, "@personal") :-
    \+ contains_any_keyword(Task, ["meeting", "standup", "sprint", "code review", "deployment", "production", "release", "ticket", "demo"]).
```

### Helper Predicates (Base Cases)

```prolog
% Extract conditional sequences (e.g., "once X is done, do Y")
extract_conditional_sequence(Input, TaskChain) :-
    contains_phrase(Input, "once"),
    split_on_phrase(Input, "once", [Condition, Action]),
    TaskChain = [dependency(Action, Condition)].

% Check if task has dependencies
has_dependencies(Task) :-
    contains_dependency_vocabulary(Task, ["blocks", "depends on", "requires", "before", "after", "then", "once"]).

% Check if task is part of sequence
is_part_of_sequence(Task) :-
    contains_dependency_vocabulary(Task, ["then", "after", "before"]).

% Check adjacency in dependency chain
adjacent_in_chain([dependency(A, B)|_], B, A).
adjacent_in_chain([_|Rest], TaskA, TaskB) :-
    adjacent_in_chain(Rest, TaskA, TaskB).

% Evaluate likelihood of sequence interpretation
% NOTE: "and" forces clarification - ambiguous (parallel vs sequential)
evaluate_sequence_likelihood(Input, Likelihood) :-
    findall(Verb, (action_verb(Verb), contains_word(Input, Verb)), Verbs),
    length(Verbs, VerbCount),
    % Only count explicit dependency connectors (then, comma)
    % "and" is excluded - triggers clarification instead
    findall(Word, (contains_word(Input, Word), member(Word, ["then", ","])), Connectors),
    length(Connectors, ConnectorCount),
    % Check for ambiguous "and" connector
    (contains_word(Input, "and") ->
        Likelihood = 0.5 ;  % Force below 0.7 threshold → clarification
        Likelihood is min(1.0, (VerbCount * 0.3 + ConnectorCount * 0.4))
    ).

% Build attributes list from components
build_attributes_list([Project, StatusTag, Tags, Dependencies], _, Result) :-
    % Combine status tag with other tags
    append([StatusTag], Tags, AllTags),
    Result = [project(Project), tags(AllTags), dependencies(Dependencies)].

% Detect vague descriptions
is_vague_description(Task) :-
    contains_any_phrase(Task, ["that thing", "the issue", "it", "this", "stuff"]).

is_vague_description(Task) :-
    \+ contains_action_verb(Task).

% Apply review mode enhancements
apply_review_enhancements(BaseAttributes, EnhancedAttributes) :-
    % In REVIEW mode, add energy and time tags using bulk operations
    % This is a placeholder - actual implementation deferred to REVIEW mode processing
    EnhancedAttributes = BaseAttributes.
```

### Dependency Inference Rules

```prolog
% Direct Dependency Detection
infer_dependency(Task, DependsOn) :-
    parse_dependency_chain(Task, Chain),
    member(dependency(Task, DependsOn), Chain).
```

## Main Inference Pipeline

```prolog
% Primary Task Structure Analysis
analyze_task_structure(Input, StructureResult) :-
    infer_task_structure(Input, "dependency_chain"),
    parse_dependency_chain(Input, Chain),
    StructureResult = [structure_type("dependency_chain"), tasks(Chain)].

analyze_task_structure(Input, StructureResult) :-
    infer_task_structure(Input, "potential_chain"),
    evaluate_sequence_likelihood(Input, Likelihood),
    (Likelihood > 0.7 -> 
        parse_dependency_chain(Input, Chain),
        StructureResult = [structure_type("inferred_chain"), tasks(Chain)] ;
        StructureResult = [structure_type("single_task"), requires_clarification(true)]
    ).

analyze_task_structure(Input, StructureResult) :-
    \+ infer_task_structure(Input, _),
    StructureResult = [structure_type("single_task")].

% Enhanced Task Processing with Dependencies
process_task_inference(TaskDescription, InferredAttributes) :-
    analyze_task_structure(TaskDescription, StructureInfo),
    member(structure_type("single_task"), StructureInfo),
    \+ member(requires_clarification(true), StructureInfo),
    % Standard single task inference
    infer_project(TaskDescription, Project),
    infer_status_tag(TaskDescription, StatusTag),
    findall(Tag, (infer_tag(TaskDescription, Tag) ; infer_quality_flag(TaskDescription, Tag)), Tags),
    findall(Dep, infer_dependency(TaskDescription, Dep), Dependencies),
    build_attributes_list([Project, StatusTag, Tags, Dependencies], TaskDescription, InferredAttributes).

process_task_inference(TaskDescription, InferredAttributes) :-
    analyze_task_structure(TaskDescription, StructureInfo),
    member(structure_type(ChainType), StructureInfo),
    member(tasks(TaskChain), StructureInfo),
    ChainType \= "single_task",
    % Multi-task dependency processing
    process_task_chain(TaskChain, TaskDescription, InferredAttributes).

% Task Chain Processing
process_task_chain(TaskChain, OriginalInput, ProcessedTasks) :-
    findall(ProcessedTask, (
        member(TaskElement, TaskChain),
        process_chain_element(TaskElement, OriginalInput, ProcessedTask)
    ), ProcessedTasks).

process_chain_element(task(TaskDesc), OriginalInput, ProcessedTask) :-
    infer_project(OriginalInput, Project),
    infer_status_tag(TaskDesc, StatusTag),
    findall(Tag, (infer_tag(OriginalInput, Tag) ; infer_quality_flag(OriginalInput, Tag)), Tags),
    % Combine status tag with other tags
    append([StatusTag], Tags, AllTags),
    ProcessedTask = task_attributes(TaskDesc, [project(Project), tags(AllTags)]).

process_chain_element(dependency(TaskA, TaskB), _, ProcessedDep) :-
    ProcessedDep = dependency_rule(TaskA, depends_on(TaskB)).

% Ambiguous Sequence Handling
process_task_inference(TaskDescription, InferredAttributes) :-
    analyze_task_structure(TaskDescription, StructureInfo),
    member(requires_clarification(true), StructureInfo),
    InferredAttributes = [
        clarification_needed(true),
        suggested_interpretation("task_sequence"),
        fallback_attributes([tags(["@personal", "next"])])
    ].

% Standard Fallback
process_task_inference(TaskDescription, DefaultAttributes) :-
    \+ infer_project(TaskDescription, _),
    \+ infer_task_structure(TaskDescription, _),
    is_vague_description(TaskDescription),
    DefaultAttributes = [tags(["@personal"])].

% Quality Flag Inference
infer_quality_flag(Task, "flag_red") :-
    is_vague_description(Task).

infer_quality_flag(Task, "flag_yellow") :-
    \+ is_vague_description(Task),
    \+ infer_project(Task, _).  % Low confidence - no project match
```

## Exclusion Rules for RECORD Mode

```prolog
% Never Infer These Attributes in RECORD Mode
forbidden_inference_record("@focus").
forbidden_inference_record("@admin").
forbidden_inference_record("@creative").
forbidden_inference_record("@review").
forbidden_inference_record("@quick").
forbidden_inference_record("@medium").
forbidden_inference_record("@long").
forbidden_inference_record("@ongoing").
forbidden_inference_record("priority").
forbidden_inference_record("annotations").

% Dependencies ARE allowed in RECORD mode when explicitly stated
allowed_inference_record("depends") :-
    dependency_explicitly_stated.
```

## Execution Logic

```prolog
% Main Processing Entry Point
execute_task_processing(Mode, TaskDescription, Result) :-
    Mode = "RECORD",
    process_task_inference(TaskDescription, Result).
```

## Usage Instructions for LLM

1. **For each incoming task description**, call `execute_task_processing(Mode, TaskDescription, Result)`
2. **Inference is only used in RECORD mode** - other modes (SYNC/REVIEW/CLEANUP/REPORT) operate on existing tasks using reporting tools
3. **Dependency Detection**: The system will automatically detect dependency vocabulary and parse task relationships
4. **Multi-task Output**: When dependencies are detected, the system returns multiple task objects with appropriate `depends` parameters
5. **Clarification Handling**: For ambiguous sequences, the system requests clarification while providing fallback interpretation
6. **Role Context**: As a task management assistant, prioritize task structure patterns over conversational interpretation
7. **Validation**: The system validates dependency chains and requires explicit task descriptions (no pronouns or vague references)
8. **Apply results** to your task management system, respecting the rule precedence
9. **Dependencies in RECORD mode** are allowed when explicitly stated in the input
10. **Semantic Reasoning**: Keyword lists in predicates are exemplary, not exhaustive. Use natural language understanding to match semantically similar concepts. Example: "pull request review" matches @work even though "pull request" isn't listed, because it's semantically similar to "code review"

### Example Processing:

```prolog
Input: "Research TaskWarrior hooks then write documentation"
Output: [
  task_attributes("Research TaskWarrior hooks", [
    project("Taskwarrior_integration"),
    tags(["next", "@personal"])
  ]),
  task_attributes("Write documentation", [
    project("Writing"),
    tags(["waiting", "@personal"]),
    depends("Research TaskWarrior hooks")
  ])
]

Input: "Fix bug after meeting with Jane"
Output: [
  task_attributes("Meeting with Jane", [
    project("Alexandria_Development"),
    tags(["next", "@work"])
  ]),
  task_attributes("Fix bug", [
    project("Alexandria_Development"),
    tags(["waiting", "@work"]),
    depends("Meeting with Jane")
  ])
]

Input: "Research hooks and write docs and test MCP"
Output: [
  clarification_needed(true),
  suggested_interpretation("task_sequence"),
  fallback_attributes([tags(["@personal", "next"])]),
  prompt("Did you mean: 3 tasks in sequence (Research → Write → Test) or 3 separate parallel tasks?")
]
```