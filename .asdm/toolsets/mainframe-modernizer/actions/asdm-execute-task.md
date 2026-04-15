# Execute Migration Task

## Description
Execute a specific migration task by reading its task detail document and following the prescribed transformation steps. This is the primary action for the Code Transformation phase — it reads a task specification (from `.asdm/workspace/planning/tasks/`) and guides the AI agent through the complete migration of that work unit, covering all relevant legacy technology layers (COBOL, Java, BMS, JCL, etc.) into the target modern architecture.

## Usage
```
/asdm-execute-task T{ID}
```

**Examples:**
```
/asdm-execute-task T0.1
/asdm-execute-task T1.1
/asdm-execute-task T3.2
```

## Prerequisites
Before executing any task, ensure the following are complete:
1. **Phase 1 (Discovery)** — `/asdm-inventory-artifacts` and `/asdm-analyze-architecture` have been run
2. **Phase 2 (Design)** — `/asdm-design-target-architecture` has been run and the target architecture document exists
3. **All dependency tasks** are completed (check the WBS dependency graph)

## Process

### Step 1: Load Task Specification
1. Locate the task detail document at `.asdm/workspace/planning/tasks/T{ID}-*.md`
2. If the detail document does not exist, generate it from the WBS overview at `.asdm/workspace/planning/03-wbs-overview.md` using the task entry as the specification source
3. Parse the task document to extract:
   - Task ID, title, and priority
   - All legacy source artifacts (COBOL programs, copybooks, JAX-RS resources, JCICS/JDBC DAOs, JZOS interfaces, z/OS Connect APIs, BMS mapsets, JCL scripts, CS/Payment interfaces)
   - Target deliverables (service methods, controllers, DTOs, JPA entities, React pages, Flyway migrations, etc.)
   - Conversion steps and validation criteria

### Step 2: Verify Dependencies
1. Check the WBS dependency graph for this task's dependencies
2. For each dependency task, verify that its target deliverables exist in `modern-banking/`
3. If any dependency is not satisfied, report the missing dependencies and stop

### Step 3: Analyze Legacy Sources
For each legacy source artifact listed in the task:
1. **COBOL programs** — Parse the PROCEDURE DIVISION, identify business logic paragraphs, map CICS commands, extract data structure usage from WORKING-STORAGE and copybooks. Apply transformation rules from the task document and [task-execution-spec.md](../specs/task-execution-spec.md)
2. **Copybooks** — Map COBOL data structures (PIC clauses, REDEFINES, OCCURS) to Java field types (`PIC X(n)` → `String`, `PIC 9(n)` → `Integer`/`Long`, `PIC 9(n)V9(m)` → `BigDecimal`, `OCCURS` → `List<T>`)
3. **JAX-RS resources** — Map HTTP methods, request/response structures, and error handling to Spring `@RestController` patterns (CICS `LINK` → `RestTemplate`/`FeignClient`, COMMAREA → `@RequestBody` DTO)
4. **JCICS/JDBC DAOs** — Map data access patterns to Spring Data JPA repositories (`CICS READ` → `findById()`, `WRITE` → `save()`, `DELETE` → `deleteById()`)
5. **JZOS interfaces** — Map data interface classes to DTOs with Bean Validation
6. **z/OS Connect APIs** — Map Swagger 2.0 operations to OpenAPI 3.0 REST endpoints (eliminate z/OS Connect service layer, call services directly)
7. **BMS mapsets** — Map screen fields to React form components (`ATTRB=UNPROT` → editable input, `ATTRB=PROT` → read-only, PF3 → back button, PF5 → submit)
8. **JCL scripts** — Map batch processing to Spring Batch or Flyway (JCL JOB → Spring Batch `Job`, EXEC STEP → Spring Batch `Step`, DDL scripts → Flyway migrations)
9. **CS/Payment interfaces** — Map Spring Boot + Thymeleaf controllers to REST APIs + React pages

### Step 4: Implement Target Deliverables
For each target deliverable specified in the task:
1. Create or update the target file in `modern-banking/` following the project structure defined in [project-structure-spec.md](../specs/project-structure-spec.md)
2. Apply the transformation rules from the relevant spec documents:
   - Service classes: `@Service` with business logic from COBOL paragraphs
   - Controllers: `@RestController` with endpoints from JAX-RS resources + z/OS Connect APIs
   - DTOs: Request/Response classes from JZOS interfaces + COMMAREA structures
   - JPA Entities: `@Entity` classes from copybook data structures
   - Repositories: `JpaRepository` interfaces from JDBC/JCICS DAO patterns
   - React pages: Page components from BMS mapsets
   - Flyway migrations: SQL scripts from Db2 DDL / VSAM schema definitions
3. Replace all placeholder `// TODO` stubs with actual migrated code
4. Ensure proper cross-service integration per [integration-spec.md](../specs/integration-spec.md)

### Step 5: Validate Task Completion
1. Verify all target deliverables exist and compile
2. Run unit tests for the migrated service/controller
3. Verify API endpoint contracts match the legacy transaction behavior
4. Check that all legacy source artifacts listed in the task have been accounted for
5. Update the task status in the WBS overview document from "Pending" to "Done"

### Step 6: Run Build Verification
1. Execute `mvn compile` in `modern-banking/` to verify no compilation errors
2. If compilation fails, fix the errors before proceeding
3. If there are test failures, document them but do not block the task completion

## Output
- All target deliverables specified in the task document, implemented in `modern-banking/`
- Updated task status in the WBS overview document
- A summary of what was migrated, including:
  - Legacy sources processed
  - Target files created/modified
  - Any deviations from the task specification
  - Known issues or TODOs remaining

## Task Document Format

Each task detail document at `.asdm/workspace/planning/tasks/T{ID}-*.md` should follow this structure:

```markdown
# Task {ID}: {Title}

> **Priority**: P0|P1|P2 | **Dependencies**: T{X.X}, T{Y.Y} | **Target Service**: {service-name}

## Legacy Sources

### COBOL Programs
- `{program}.cbl` — {description of business logic}

### Copybooks
- `{copybook}.cpy` — {data structure description}

### Java (JAX-RS)
- `{Resource}.java` — {HTTP methods and endpoints}

### Java (JCICS/JDBC)
- `{DAO}.java` — {data access patterns}

### Java (JZOS Data Interfaces)
- `{Interface}.java` — {data structure mapping}

### z/OS Connect
- `{api}` API + `{service}` service — {operation description}

### BMS Mapsets
- `{MAPSET}.bms` — {screen description}

### JCL Scripts
- `{SCRIPT}.jcl` — {batch operation description}

### CS/Payment Interface
- `{Controller}.java` — {interface description}

## Target Deliverables

### Service Layer
- `{Service}.java` — {method signatures}

### Controller Layer
- `{Controller}.java` — {endpoints}

### DTOs
- `{Request}.java`, `{Response}.java` — {fields}

### JPA Entities / Repositories
- `{Entity}.java`, `{Repository}.java` — {if not created in T0.1}

### Flyway Migrations
- `V{N}__{description}.sql` — {if schema changes needed}

### React Pages
- `{Page}.tsx` — {form fields and actions}

## Conversion Steps
1. {Step 1}
2. {Step 2}
...

## Validation Criteria
- [ ] {Criterion 1}
- [ ] {Criterion 2}
...
```

## Related Specifications
- [task-execution-spec.md](../specs/task-execution-spec.md) — Rules for task document format and execution workflow (includes transformation rules for all legacy technology layers)
- [project-structure-spec.md](../specs/project-structure-spec.md) — Target directory structure and naming
- [integration-spec.md](../specs/integration-spec.md) — Service integration patterns
