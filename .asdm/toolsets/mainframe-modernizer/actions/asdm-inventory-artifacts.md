# Inventory Artifacts

## Description
Scan and catalog all legacy artifacts in the CBSA (CICS Banking Sample Application) directory. This action produces a complete inventory of COBOL programs, JCL scripts, BMS mapsets, Java programs, copybooks, and other artifacts with their classifications and relationships.

## Usage
```
/asdm-inventory-artifacts
```

## Process
1. Walk through `cics-banking-sample-application-cbsa/` directory tree
2. Identify and classify each artifact by type (COBOL, JCL, BMS, Java, Copybook, etc.)
3. Extract metadata from each artifact (program name, CICS resources referenced, data structures used)
4. Identify inter-artifact dependencies (CALL statements, COPY statements, JCL PROC includes)
5. Produce a structured inventory report

## Output
A structured inventory report containing:
- Complete list of artifacts with type, path, and classification
- Dependency graph between artifacts
- CICS resource usage summary
- Data structure catalog

## Related Specifications
See [inventory-spec.md](../specs/inventory-spec.md) for detailed specifications.
