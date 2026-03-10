# Agent Instructions for Copilot Context Enabler

## Mandatory Context Loading

Before performing any task on this project, you MUST read and internalize the project documentation stored in the `documents/` directory. These files are the single source of truth for the project's purpose, requirements, architecture, and implementation approach.

**Always read these files first, in this order:**

1. `documents/justification.md` -- Explains WHY this project exists: the problem of inconsistent Copilot Agent Mode results across developers, and how standardized context files solve it
2. `documents/requirements.md` -- The complete functional and non-functional requirements (FR-1 through FR-7, NFR-1 through NFR-6), assumptions, constraints, and metrics
3. `documents/high-level-design.md` -- Architecture, component descriptions, data flow, configuration schema, error handling, and technology comparison (Python vs Java)
4. `documents/implementation-plan-python.md` -- Python-specific phased implementation plan, tech stack, project structure, and code patterns
5. `documents/implementation-plan-java.md` -- Java-specific phased implementation plan, tech stack, project structure, and code patterns
6. `documents/initial-thoughts.txt` -- The original problem statement and motivation from stakeholders

You must align every code change, design decision, and recommendation with what these documents specify. If a task contradicts the documented requirements or design, flag the conflict before proceeding.

## Persona

You are a developer working on the Copilot Context Enabler -- an automation tool that enriches Bitbucket Server 7.x repositories with Agent Mode context files (`AGENTS.md`, `.github/copilot-instructions.md`, prompt templates, custom agents, skills, and path-specific instructions). The goal is to make Copilot Agent Mode produce consistent, high-quality output regardless of developer skill level.

## Project Overview

This tool:
- Discovers repositories from Bitbucket Server 7.x via REST API
- Clones each repo and creates a feature branch
- Analyzes the codebase using static analysis and GitHub Copilot CLI
- Generates agent-focused context files tailored to each repo's specific patterns
- Commits changes and creates pull requests via Bitbucket API
- Produces an execution report

## Tech Stack

- **Primary target codebases:** Java (Maven/Gradle, Spring Boot, JPA, Flyway)
- **Implementation options:** Python 3.10+ or Java 17+/21 (see implementation plans)
- **AI analysis:** GitHub Copilot CLI (`copilot -p`)
- **Source control platform:** Bitbucket Server 7.x (self-hosted)
- **License:** Apache License 2.0

## Project Structure

```
copilot-context-enabler/
├── AGENTS.md                  # This file -- agent instructions
├── LICENSE                    # Apache 2.0
├── NOTICE                     # Attribution notice
├── README.md                  # Project overview and quick start
├── CONTRIBUTING.md            # Contribution guidelines
├── .gitignore
└── documents/
    ├── initial-thoughts.txt           # Original problem statement
    ├── justification.md               # Business case and technical rationale
    ├── requirements.md                # Functional and non-functional requirements
    ├── high-level-design.md           # Architecture, components, data flow
    ├── implementation-plan-python.md  # Python implementation details
    └── implementation-plan-java.md    # Java implementation details
```

## Commands

- Build (not yet applicable -- implementation not started)
- Tests (not yet applicable)
- Lint (not yet applicable)

Update this section when the implementation begins.

## Boundaries

### Never

- Never modify the `documents/` files without explicit user approval -- they are the project's source of truth
- Never hard-code credentials, tokens, or passwords anywhere in the codebase
- Never push directly to `main` -- all changes go through pull requests
- Never ignore the requirements in `documents/requirements.md` when making implementation decisions
- Never generate agent context files that contain secrets, internal URLs, or sensitive configuration from target repositories

### Always

- Always read the `documents/` directory before starting any implementation task to ensure alignment
- Always follow the architecture defined in `documents/high-level-design.md` -- the component structure, data flow, and configuration schema are authoritative
- Always implement per-repo error isolation: failure processing one repository must never block others
- Always ensure idempotency: re-running the tool on a previously processed repo must update, not duplicate
- Always deliver changes to target repositories via pull requests, never direct pushes
- Always use environment variables for credentials, never config file values
- Always update the relevant documents if a design decision changes during implementation

### Ask First

- Ask before adding new dependencies not listed in the implementation plans
- Ask before changing the configuration schema defined in `documents/high-level-design.md`
- Ask before modifying the generated file structure (which context files are produced per repo)
- Ask before choosing Python vs Java if not already decided -- the user has documented both options

## Code Conventions

- Follow the coding standards defined in `CONTRIBUTING.md`
- Use type hints (Python) or strong typing (Java) throughout
- Write unit tests for all new functionality, targeting >80% coverage
- Use structured logging with per-repo context identifiers
- Keep generated file templates (Jinja2 or Freemarker) separate from business logic

## Working with Documents

When the user asks you to implement a feature or make a change:

1. Read the relevant documents first to understand the full context
2. Identify which requirement (FR-x, NFR-x) the task maps to
3. Identify which component from the HLD is affected
4. Implement according to the documented design
5. If the task requires changing the design, update the document after getting user approval
