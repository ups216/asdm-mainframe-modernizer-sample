# List Migration Tasks

## Description
List all available migration tasks from the WBS (Work Breakdown Structure), showing their status, priority, dependencies, and execution wave. Use this action to understand the full scope of migration work and decide which task to execute next.

## Usage
```
/asdm-list-tasks [optional: --wave N | --group GROUP | --status STATUS | --priority P0|P1|P2]
```

## Process
1. Read the WBS overview document at `.asdm/workspace/planning/03-wbs-overview.md` (or the WBS document specified in the workspace)
2. Parse the Task Index table to extract all tasks with their ID, scenario, title, priority, dependencies, and status
3. If a filter argument is provided, apply it:
   - `--wave N` — show only tasks in execution wave N
   - `--group GROUP` — show only tasks in the specified group (e.g., "Customer Management")
   - `--status STATUS` — show only tasks with the given status (Pending, In Progress, Done)
   - `--priority P0|P1|P2` — show only tasks at the given priority level
4. Display the filtered task list in a formatted table
5. For each task, indicate whether a detail document exists at `.asdm/workspace/planning/tasks/T{ID}-*.md`

## Output
A formatted task list containing:
- Task ID, title, priority, dependencies, and current status
- Execution wave assignment
- Whether a detail document exists for the task
- Suggested next task based on dependency resolution (all dependencies satisfied) and priority

## Example
```
/asdm-list-tasks --priority P0

Task ID | Scenario | Title                   | Priority | Dependencies | Status  | Detail Doc
--------|----------|-------------------------|----------|-------------|---------|----------
T0.1    | 0.1      | Data Layer Foundation   | P0       | —           | Pending | Yes
T1.1    | 1.1      | Create Customer         | P0       | T0.1        | Pending | Yes
T1.2    | 1.2      | Inquire Customer        | P0       | T0.1        | Pending | Yes
T2.1    | 2.1      | Create Account          | P0       | T1.1        | Pending | Yes
T2.2    | 2.2      | Inquire Account         | P0       | T0.1        | Pending | Yes
T3.1    | 3.1      | Credit / Debit          | P0       | T2.2        | Pending | Yes
T3.2    | 3.2      | Transfer                | P0       | T2.2, T3.1  | Pending | Yes
T5.2    | 5.2      | Named Counter Sync      | P0       | T1.1, T2.1  | Pending | Yes
T5.5    | 5.5      | BMS → React SPA         | P0       | T1.1–T4.2   | Pending | Yes
T7.1    | 7.1      | E2E Integration         | P0       | T1.1–T6.1   | Pending | Yes

Suggested next: T0.1 (no unmet dependencies)
```

## Related Specifications
See [task-execution-spec.md](../specs/task-execution-spec.md) for task document format and execution rules.
