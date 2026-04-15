# Inventory Specification

## Purpose
This document defines the rules, constraints, and expected outputs for the artifact inventory process.

## Artifact Classification

### Supported Artifact Types
| Type | File Extension | Description |
|------|---------------|-------------|
| COBOL Program | `.cbl`, `.cob` | Main business logic programs |
| COBOL Copybook | `.cpy`, `.copy` | Reusable data structure definitions |
| JCL Job | `.jcl` | Batch job control language scripts |
| BMS Mapset | `.bms` | 3270 screen map definitions |
| Java Program | `.java` | CICS Java programs (JCICS API) |
| CICS Resource | `.csd`, `.jcl` (CICS defs) | CICS resource definitions |

### Inventory Rules
1. Every file in the CBSA directory must be classified
2. Unrecognized file types should be flagged as "UNKNOWN" with path and content hints
3. Duplicate artifacts (same program name, different paths) must be identified
4. Nested directory structures must be flattened in the inventory with full path preserved

## Output Format

The inventory report must be a structured document containing:

```json
{
  "summary": {
    "total_artifacts": "number",
    "by_type": { "COBOL": "number", "JCL": "number", ... },
    "scan_timestamp": "ISO 8601 datetime"
  },
  "artifacts": [
    {
      "id": "unique identifier",
      "name": "program/file name",
      "type": "COBOL|JCL|BMS|JAVA|COPYBOOK|UNKNOWN",
      "path": "relative path from CBSA root",
      "description": "brief description or TITLE clause",
      "cics_resources": ["list of CICS resources referenced"],
      "dependencies": ["list of other artifact IDs this depends on"]
    }
  ]
}
```

## Constraints
- Inventory must be deterministic — running twice on the same source must produce identical results
- Large files (>1MB) should be sampled, not fully parsed, with a flag indicating incomplete analysis
- Binary files (images, compiled objects) should be listed but not parsed
