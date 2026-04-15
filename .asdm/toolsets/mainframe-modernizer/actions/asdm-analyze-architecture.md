# Analyze Architecture

## Description
Analyze the legacy CBSA architecture by mapping inter-program calls, CICS resource references, data flow patterns, and transaction boundaries. This action produces a comprehensive architecture map that serves as the foundation for designing the target system.

## Usage
```
/asdm-analyze-architecture
```

## Process
1. Parse COBOL programs for CICS commands (READ, WRITE, START, LINK, XCTL, etc.)
2. Map program-to-program call chains (LINK/XCTL relationships)
3. Identify transaction boundaries (CICS transaction IDs and associated programs)
4. Analyze data access patterns (VSAM files, DB2 tables, TDQ/TSQ usage)
5. Map BMS mapsets to their associated programs and transactions
6. Identify shared data structures and communication areas
7. Produce architecture dependency map

## Output
- Program call graph (which programs call which)
- Transaction-to-program mapping
- Data access pattern summary
- Resource dependency matrix
- Business function grouping recommendations

## Related Specifications
See [architecture-analysis-spec.md](../specs/architecture-analysis-spec.md) for detailed specifications.
