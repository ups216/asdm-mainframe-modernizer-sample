# Task Execution Specification

## Purpose
This document defines the rules, constraints, and expected format for task-driven migration execution. Each task in the WBS (Work Breakdown Structure) is a self-contained work unit that migrates a complete business scenario across all relevant legacy technology layers.

## Task Document Format

### Location
Task detail documents are stored in the project workspace at:
```
.asdm/workspace/planning/tasks/T{ID}-{kebab-case-title}.md
```

### Required Sections
Every task document must contain the following sections:

| Section | Required | Description |
|---------|----------|-------------|
| **Header** | Yes | Task ID, title, priority, dependencies, target service |
| **Legacy Sources** | Yes | Exhaustive list of all legacy artifacts for this task, organized by technology layer |
| **Target Deliverables** | Yes | All files to be created or modified in `modern-banking/`, organized by architectural layer |
| **Conversion Steps** | Yes | Ordered list of transformation steps the AI agent must follow |
| **Validation Criteria** | Yes | Checklist of conditions that must be true for the task to be considered complete |

### Header Format
```markdown
# Task {ID}: {Title}

> **Priority**: P0|P1|P2 | **Dependencies**: T{X.X}, T{Y.Y} | **Target Service**: {service-name}
```

### Legacy Sources Subsections
Organize legacy sources by technology layer. Include only layers relevant to the task:

| Subsection | When Present |
|------------|-------------|
| COBOL Programs | Task involves COBOL business logic conversion |
| Copybooks | Task uses COBOL data structures |
| Java (JAX-RS) | Task has REST API resources to migrate |
| Java (JCICS/JDBC) | Task has CICS file access or Db2 data access |
| Java (JZOS Data Interfaces) | Task has data interface classes |
| z/OS Connect | Task has z/OS Connect API operations |
| BMS Mapsets | Task has terminal screen maps |
| JCL Scripts | Task has batch processing scripts |
| CS/Payment Interface | Task has Customer Services or Payment interface code |

Each source entry must include:
- File name with extension
- Brief description of what it contains or does

### Target Deliverables Subsections
Organize by architectural layer:

| Subsection | Contents |
|------------|----------|
| Service Layer | `@Service` classes with method signatures |
| Controller Layer | `@RestController` classes with endpoint mappings |
| DTOs | Request/Response classes with key fields |
| JPA Entities / Repositories | Entity classes and repository interfaces (if not created in T0.1) |
| Flyway Migrations | SQL migration scripts (if schema changes needed) |
| React Pages | Page components with form fields and actions |

## Execution Rules

### Dependency Resolution
1. A task MUST NOT be executed until all its dependency tasks are completed
2. "Completed" means: all target deliverables of the dependency task exist and compile in `modern-banking/`
3. If a dependency is not satisfied, the AI agent MUST stop and report the missing dependency

### Transformation Compliance
1. All code transformations MUST follow the rules in the corresponding technology-layer spec:
   - COBOL programs → copybook data structure mapping rules (PIC X → String, PIC 9 → Integer/Long, PIC 9V9 → BigDecimal, REDEFINES → @Transient, OCCURS → List<T>); one @Service per COBOL program; business logic preserved exactly
   - BMS mapsets → each BMS mapset → one React page; ATTRB=UNPROT → editable input; ATTRB=PROT → read-only; PF3 → back button; PF5 → submit
   - CICS Java programs → JCICS Program.link() → RestTemplate/FeignClient; File.read/write → JpaRepository; COMMAREA → @RequestBody DTO; CICS error codes → HTTP status codes
   - JCL scripts → JCL JOB → Spring Batch Job; EXEC STEP → Spring Batch Step; DDL scripts → Flyway migrations; deployment JCL → GitHub Actions
2. Target file structure MUST follow [project-structure-spec.md](project-structure-spec.md)
3. Service integration MUST follow [integration-spec.md](integration-spec.md)

### Task Scope
1. A task MUST implement ALL legacy sources listed in the task document — no partial migration
2. A task MUST NOT modify files owned by another service without explicit cross-service API calls
3. A task MAY update shared common module classes if needed (DTOs, exceptions, utilities)
4. A task MUST replace existing `// TODO` placeholder code in target files rather than creating duplicate files

### Validation Requirements
1. After execution, `mvn compile` MUST succeed in `modern-banking/`
2. All validation criteria in the task document MUST be satisfied
3. The task status in the WBS overview MUST be updated from "Pending" to "Done"

## Task Execution Workflow

```
1. Load task document → .asdm/workspace/planning/tasks/T{ID}-*.md
2. Verify dependencies → check all dependency tasks are "Done"
3. Analyze legacy sources → read each source file in cics-banking-sample-application-cbsa/
4. Implement target deliverables → create/modify files in modern-banking/
5. Validate completion → compile, test, check criteria
6. Update WBS status → mark task as "Done" in 03-wbs-overview.md
```

## Constraints
- Task documents are project-specific and live in the workspace, NOT in the toolset directory
- The toolset provides the execution framework; the workspace provides the task specifications
- Task documents may be generated from the WBS overview if detail documents don't exist yet
- Each task should be executable independently (given its dependencies are met)
- Tasks should be atomic — either fully complete or not at all
