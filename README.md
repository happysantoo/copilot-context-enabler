# Copilot Context Enabler

An automation tool that enriches source code repositories with GitHub Copilot **Agent Mode** context files -- `AGENTS.md`, custom agents, reusable prompt templates, skills, and path-specific instructions -- so that Copilot Agent Mode produces consistent, high-quality output regardless of developer skill level.

## The Problem

GitHub Copilot Agent Mode produces inconsistent results across developers. A senior developer who understands the codebase writes detailed prompts and gets great output. A junior developer writes vague prompts and gets off-pattern or incorrect code. The model is the same; the difference is in the context available to the agent.

## The Solution

This tool automatically analyzes each repository's codebase and generates agent-focused context files that give Copilot deep project knowledge before any developer says anything. The generated files are committed to a feature branch and delivered via pull request for team review.

### Generated Files

| File | Purpose |
|------|---------|
| `AGENTS.md` | Agent persona, build/test/lint commands, project structure, boundaries |
| `.github/copilot-instructions.md` | Repository-wide coding standards and conventions |
| `.github/instructions/*.instructions.md` | Path-specific rules (e.g., test conventions, controller patterns) |
| `.github/prompts/*.prompt.md` | Reusable task templates (e.g., "add REST endpoint", "write integration tests") |
| `.github/agents/*.agent.md` | Custom agent profiles (test-specialist, api-reviewer, migration-helper) |
| `.github/skills/**/SKILL.md` | Multi-step workflow runbooks |
| `.vscode/settings.json` | Agent-related VS Code settings |
| `.vscode/extensions.json` | Recommended extensions |

## How It Works

1. **Discover** repositories from Bitbucket Server 7.x via REST API
2. **Clone** each repository and create a feature branch
3. **Analyze** the codebase -- detect language, frameworks, build tools, project structure, patterns
4. **Generate** context files using GitHub Copilot CLI for AI-assisted analysis combined with deterministic static analysis
5. **Commit** all generated files, push the branch, and create a pull request
6. **Report** on what was processed, generated, and any errors encountered

## Features

- Targets **Copilot Agent Mode** (autonomous multi-step task execution), not just inline completions
- **Conditional generation** -- only creates files relevant to detected patterns (e.g., migration skills only if Flyway is detected)
- **Idempotent** -- safe to re-run; updates existing files without duplication
- **Concurrent processing** -- configurable thread pool for parallel repo processing
- **Resilient** -- failures on one repo don't block others
- **Configurable** -- YAML-based configuration for all behavior
- **Dry-run mode** -- preview what would be generated without committing

## Prerequisites

- Bitbucket Server 7.x with REST API access
- [GitHub Copilot CLI](https://docs.github.com/en/copilot/github-copilot-in-the-cli) installed and authenticated
- Git with SSH or HTTPS access to target repositories

## Quick Start

```bash
# Clone this repository
git clone https://github.com/your-org/copilot-context-enabler.git
cd copilot-context-enabler

# Copy and edit the configuration
cp config.example.yaml config.yaml
# Edit config.yaml with your Bitbucket URL, target projects, etc.

# Set credentials
export BITBUCKET_USER="your-username"
export BITBUCKET_TOKEN="your-personal-access-token"

# Run (see implementation plans for language-specific instructions)
```

## Configuration

```yaml
bitbucket:
  base_url: "https://bitbucket.company.com"
  auth_type: "token"

targets:
  - project_key: "MYPROJECT"
    repos: []  # empty = all repos in project

branch:
  name: "copilot-context/enhance"
  base: "main"

processing:
  concurrency: 4

generation:
  agents_md: true
  copilot_instructions: true
  path_instructions: true
  prompt_templates: true
  custom_agents: true
  skills: true
  vscode_config: true
```

See [`config.example.yaml`](config.example.yaml) for the full configuration reference.

## Documentation

| Document | Description |
|----------|-------------|
| [Justification](documents/justification.md) | Why agent context files solve the consistency problem across developers |
| [Requirements](documents/requirements.md) | Functional and non-functional requirements |
| [High-Level Design](documents/high-level-design.md) | Architecture, components, data flow, and technology comparison |
| [Implementation Plan (Python)](documents/implementation-plan-python.md) | Python implementation: phased plan, tech stack, project structure |
| [Implementation Plan (Java)](documents/implementation-plan-java.md) | Java implementation: phased plan, tech stack, project structure |

## Contributing

We welcome contributions. Please see [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines.

## License

This project is licensed under the Apache License 2.0 -- see the [LICENSE](LICENSE) file for details.

```
Copyright 2026 Copilot Context Enabler Contributors

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
```
