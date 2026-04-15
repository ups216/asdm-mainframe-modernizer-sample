# Architecture Analysis Specification

## Purpose
This document defines the rules and constraints for analyzing the legacy CBSA architecture, mapping dependencies, and identifying business function boundaries.

## Analysis Dimensions

### 1. Program Call Graph
- Map all `EXEC CICS LINK` and `EXEC CICS XCTL` relationships
- Include static call chains (COBOL CALL statements)
- Identify entry points (programs started by CICS transactions)

### 2. CICS Resource Mapping
- VSAM file usage (READ, WRITE, REWRITE, DELETE)
- TDQ (Transient Data Queue) usage
- TSQ (Temporary Storage Queue) usage
- DB2 access patterns
- CICS program-to-transaction mapping

### 3. Data Flow Analysis
- Trace data elements through program call chains
- Identify shared communication areas (COMMAREA, CHANNELS)
- Map copybook-level data structure sharing

### 4. Transaction Boundary Identification
- CICS transaction IDs and their entry programs
- PCT (Program Control Table) definitions
- Transaction isolation and commit boundaries

## Output Format

```json
{
  "call_graph": {
    "nodes": [{ "id": "program_name", "type": "COBOL|JAVA" }],
    "edges": [{ "from": "caller", "to": "callee", "type": "LINK|XCTL|CALL" }]
  },
  "resource_usage": {
    "files": [{ "name": "VSAM_FILE", "accessed_by": ["program1", "program2"] }],
    "queues": [{ "name": "QUEUE_NAME", "type": "TDQ|TSQ", "accessed_by": [] }]
  },
  "transactions": [
    { "transid": "ABCD", "entry_program": "PGM001", "programs": [] }
  ],
  "data_flows": [
    { "from": "program1", "to": "program2", "data_structure": "COPYBOOK-NAME" }
  ],
  "business_functions": [
    { "name": "suggested_function_name", "programs": [], "resources": [] }
  ]
}
```

## Constraints
- Circular dependencies must be flagged explicitly
- Missing resources (referenced but not found) must be listed as warnings
- Business function grouping should follow DDD bounded context heuristics
