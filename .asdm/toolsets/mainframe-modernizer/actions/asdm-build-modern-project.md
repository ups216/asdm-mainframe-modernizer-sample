# Build Modern Project

## Description
Assemble the complete modernized project structure in the `modern-banking/` directory. This action creates the proper directory layout, build files, configuration, and integrates all transformed components into a cohesive project.

## Usage
```
/asdm-build-modern-project
```

## Process
1. Create the target directory structure under `modern-banking/`
2. Generate Maven/Gradle build files (pom.xml or build.gradle)
3. Create Spring Boot application entry point and configuration
4. Organize transformed code into proper packages (controllers, services, repositories, models)
5. Set up frontend project structure (package.json, build config)
6. Create Docker and Kubernetes deployment manifests
7. Generate README and documentation for the modernized project

## Output
- Complete `modern-banking/` project directory with:
  - Backend: Spring Boot multi-module project
  - Frontend: React/Vue application
  - Docker/K8s deployment configurations
  - Documentation and README

## Related Specifications
See [project-structure-spec.md](../specs/project-structure-spec.md) for detailed specifications.
