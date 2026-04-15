# Mainframe Modernizer Toolset Installation

**Toolset ID:** `mainframe-modernizer`

## Overview
This document provides instructions for installing and setting up the Mainframe Modernizer toolset in a workspace. Mainframe Modernizer provides a structured set of actions for modernizing IBM CICS mainframe applications into cloud-native microservices architectures. The toolset is general-purpose — project-specific task definitions live in the workspace (`.asdm/workspace/planning/tasks/`).

## AI Guided Installation
To install this toolset using AI Guided Installation, copy and paste the following prompt into your AI Coding tool's chat window:

```shell
Follow instructions in .asdm/toolsets/mainframe-modernizer/INSTALL.md
```

## Prerequisites

- Source code: legacy application directory present in workspace
- Target directory: `modern-banking/` (will be created if not present)
- Java 17+ and Maven for backend services
- Node.js 18+ for frontend application
- Docker for containerized development

## Installation Steps

### 1. Detect the current `Agentic Engine` provider

Detect the current AI coding assistant provider (e.g., Claude Code, GitHub Copilot, Tencent CodeBuddy). Using the following guidelines to detect the provider:

- If `.claude` directory exists, use `Claude Code`
- If `.github` directory exists, use `GitHub Copilot`
- If `.codebuddy` directory exists, use `Tencent CodeBuddy`
- If no such folder is found in the current workspace, give user a prompt to select a provider manually

### 2. Create shortcut commands for Mainframe Modernizer (toolset ID: `mainframe-modernizer`) in provider's entry point

Create shortcut commands in the appropriate location based on the detected provider:

#### For Claude Code (`.claude/commands/`):
Claude Code uses Markdown files with Frontmatter metadata for slash commands. Create commands by concatenating Claude-specific frontmatter with instruction content:

```bash
mkdir -p .claude/commands/

# Inventory Artifacts command
cat > .claude/commands/asdm-inventory-artifacts.md << 'EOF'
---
description: "Scan and catalog all legacy artifacts"
argument-hint: "[optional: specific subdirectory to focus on]"
---

EOF
cat .asdm/toolsets/mainframe-modernizer/actions/asdm-inventory-artifacts.md >> .claude/commands/asdm-inventory-artifacts.md

# Analyze Architecture command
cat > .claude/commands/asdm-analyze-architecture.md << 'EOF'
---
description: "Analyze legacy architecture and map dependencies"
argument-hint: "[optional: specific area to focus on]"
---

EOF
cat .asdm/toolsets/mainframe-modernizer/actions/asdm-analyze-architecture.md >> .claude/commands/asdm-analyze-architecture.md

# Design Target Architecture command
cat > .claude/commands/asdm-design-target-architecture.md << 'EOF'
---
description: "Design the target cloud-native microservices architecture"
argument-hint: "[optional: architecture style preference]"
---

EOF
cat .asdm/toolsets/mainframe-modernizer/actions/asdm-design-target-architecture.md >> .claude/commands/asdm-design-target-architecture.md

# List Tasks command
cat > .claude/commands/asdm-list-tasks.md << 'EOF'
---
description: "List migration tasks from the WBS with status and dependencies"
argument-hint: "[optional: --wave N | --group GROUP | --status STATUS | --priority P0|P1|P2]"
---

EOF
cat .asdm/toolsets/mainframe-modernizer/actions/asdm-list-tasks.md >> .claude/commands/asdm-list-tasks.md

# Execute Task command
cat > .claude/commands/asdm-execute-task.md << 'EOF'
---
description: "Execute a specific migration task by reading its task detail document"
argument-hint: "T{ID} (e.g., T0.1, T1.1, T3.2)"
---

EOF
cat .asdm/toolsets/mainframe-modernizer/actions/asdm-execute-task.md >> .claude/commands/asdm-execute-task.md

# Build Modern Project command
cat > .claude/commands/asdm-build-modern-project.md << 'EOF'
---
description: "Assemble the modernized project structure"
argument-hint: "[optional: specific modules to build]"
---

EOF
cat .asdm/toolsets/mainframe-modernizer/actions/asdm-build-modern-project.md >> .claude/commands/asdm-build-modern-project.md

# Integrate Services command
cat > .claude/commands/asdm-integrate-services.md << 'EOF'
---
description: "Wire up microservice communication and integration"
argument-hint: "[optional: specific integration pattern to apply]"
---

EOF
cat .asdm/toolsets/mainframe-modernizer/actions/asdm-integrate-services.md >> .claude/commands/asdm-integrate-services.md

# Validate Migration command
cat > .claude/commands/asdm-validate-migration.md << 'EOF'
---
description: "Validate functional equivalence between legacy and modernized systems"
argument-hint: "[optional: specific transactions to validate]"
---

EOF
cat .asdm/toolsets/mainframe-modernizer/actions/asdm-validate-migration.md >> .claude/commands/asdm-validate-migration.md
```

#### For GitHub Copilot (`.github/prompts/`):
GitHub Copilot uses `.prompt.md` files with YAML frontmatter. Create prompt files by concatenating GitHub-specific frontmatter with instruction content:

```bash
mkdir -p .github/prompts/

# Inventory Artifacts prompt
cat > .github/prompts/asdm-inventory-artifacts.prompt.md << 'EOF'
---
agent: 'agent'
description: 'Scan and catalog all legacy artifacts'
argument-hint: 'Enter optional subdirectory or leave blank'
---

EOF
cat .asdm/toolsets/mainframe-modernizer/actions/asdm-inventory-artifacts.md >> .github/prompts/asdm-inventory-artifacts.prompt.md

# (Repeat the same pattern for all other actions...)
```

#### For Tencent CodeBuddy (`.codebuddy/commands/`):
CodeBuddy doesn't support frontmatter, so simply copy the instruction files as-is:

```bash
mkdir -p .codebuddy/commands/

cp .asdm/toolsets/mainframe-modernizer/actions/asdm-inventory-artifacts.md .codebuddy/commands/
cp .asdm/toolsets/mainframe-modernizer/actions/asdm-analyze-architecture.md .codebuddy/commands/
cp .asdm/toolsets/mainframe-modernizer/actions/asdm-design-target-architecture.md .codebuddy/commands/
cp .asdm/toolsets/mainframe-modernizer/actions/asdm-list-tasks.md .codebuddy/commands/
cp .asdm/toolsets/mainframe-modernizer/actions/asdm-execute-task.md .codebuddy/commands/
cp .asdm/toolsets/mainframe-modernizer/actions/asdm-build-modern-project.md .codebuddy/commands/
cp .asdm/toolsets/mainframe-modernizer/actions/asdm-integrate-services.md .codebuddy/commands/
cp .asdm/toolsets/mainframe-modernizer/actions/asdm-validate-migration.md .codebuddy/commands/
```

### 3. Manual Usage for Other Providers

If your AI coding assistant provider is not detected, you can directly use the instruction files by pasting the file path into your AI tool's chat:

```
Follow the instructions in .asdm/toolsets/mainframe-modernizer/actions/asdm-execute-task.md
```

## Available Commands

Once installed, the following slash commands are available:

| Command | Phase | Description |
|---------|-------|-------------|
| `/asdm-inventory-artifacts` | Discovery | Scan and catalog legacy artifacts |
| `/asdm-analyze-architecture` | Discovery | Analyze legacy architecture and dependencies |
| `/asdm-design-target-architecture` | Design | Design target cloud-native architecture |
| `/asdm-list-tasks` | Transformation | List migration tasks from WBS |
| `/asdm-execute-task T{ID}` | Transformation | Execute a specific migration task |
| `/asdm-build-modern-project` | Assembly | Build the modern-banking/ project |
| `/asdm-integrate-services` | Assembly | Wire up microservice integration |
| `/asdm-validate-migration` | Validation | Verify functional equivalence |

## Task Documents

Task documents are project-specific and must be created in the workspace at `.asdm/workspace/planning/tasks/`. They are not part of the toolset installation. The WBS overview at `.asdm/workspace/planning/03-wbs-overview.md` defines the task list; detail documents can be generated on demand when `/asdm-execute-task` is run for a task that doesn't have one yet.

## Verification

After installation, verify that:
1. Shortcut commands are created in the appropriate provider directory
2. The toolset files are located in `.asdm/toolsets/mainframe-modernizer/`
3. The WBS overview document exists at `.asdm/workspace/planning/03-wbs-overview.md`
4. Task documents exist at `.asdm/workspace/planning/tasks/` (or can be generated)

## License

Copyright (c) 2026 LeansoftX.com & iSoftStone. All rights reserved.

Licensed under the PROPRIETARY SOFTWARE LICENSE. See [LICENSE](LICENSE) in the project root for license information.
