# Tools Directory

This directory contains utility tools that support the Mainframe Modernizer toolset functionality. These tools are designed to be called by agents in CLI environments during the modernization process.

## Planned Tools

| Tool | Description | Status |
|------|-------------|--------|
| `cobol-parser` | Parse COBOL source files and extract structure, paragraphs, and CICS commands | Planned |
| `jcl-analyzer` | Parse JCL jobs and extract step definitions, DD allocations, and PROC calls | Planned |
| `bms-mapset-parser` | Parse BMS mapset definitions and extract field layouts and attributes | Planned |
| `cics-resource-mapper` | Map CICS resource definitions to modern service configurations | Planned |
| `migration-validator` | Compare legacy and modernized outputs for functional equivalence | Planned |

## Structure

Each tool should have its own subdirectory with:
- `README.md`: Tool documentation
- `main.py` or `main.js`: Entry point
- `config.json`: Configuration (optional)
- `tests/`: Test suite (recommended)

## Guidelines

1. Tools should be self-contained and independently executable
2. Each tool should have a single responsibility
3. Configuration should be externalized
4. Error handling should be robust
5. Logging should be configurable
6. Tools should produce structured output (JSON) for programmatic consumption
