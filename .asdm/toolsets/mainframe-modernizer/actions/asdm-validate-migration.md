# Validate Migration

## Description
Verify functional equivalence between the legacy CBSA system and the modernized banking application. This action creates test suites and validation procedures to ensure the modernized system faithfully reproduces the legacy system's business logic and behavior.

## Usage
```
/asdm-validate-migration
```

## Process
1. Generate unit tests for each transformed service/component
2. Create integration tests for end-to-end business flows
3. Build test data sets that mirror legacy test scenarios
4. Compare legacy transaction outputs with modernized API responses
5. Validate data integrity across migration boundaries
6. Produce a migration validation report with coverage metrics

## Output
- Unit test suites for all services
- Integration test suites for business flows
- Test data fixtures
- Migration validation report
- Coverage and equivalence metrics

## Related Specifications
See [validation-spec.md](../specs/validation-spec.md) for detailed specifications.
