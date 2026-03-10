<!--
Copyright 2026 Copilot Context Enabler Contributors
SPDX-License-Identifier: Apache-2.0
-->

# Requirements Document: Copilot Context Enabler

**Version:** 1.0
**Status:** Draft
**Target:** Copilot Agent Mode Consistency Across Developers

---

## 1. Problem Statement

The organization maintains Java applications across a self-hosted Bitbucket Server 7.x instance, organized under projects and repositories. Developers use Visual Studio Code with GitHub Copilot, primarily leveraging **Agent Mode** for autonomous multi-step coding tasks.

Agent Mode produces inconsistent results across developers of different skill levels. A senior developer who understands the codebase writes detailed, context-rich prompts and receives high-quality output. A junior developer writes vague prompts and receives off-pattern or incorrect code. The root cause is not the AI model -- it is the absence of standardized project context files in repositories that the agent can use as baseline knowledge.

There is no scalable, automated mechanism to generate and maintain these context files across the organization's repositories.

---

## 2. Goals and Success Criteria

### Primary Goals

| ID | Goal | Success Criterion |
|----|------|-------------------|
| G-1 | Consistent agent task execution | Agent-generated code follows project patterns regardless of who prompted it |
| G-2 | Reduced skill-level dependency | Junior developers achieve agent output quality comparable to senior developers |
| G-3 | Reusable task workflows | Common development tasks are available as prompt templates, not ad-hoc prompts |
| G-4 | Specialized agent roles | Custom agents exist for testing, review, and migration tasks |
| G-5 | Zero manual effort per repo | Context file generation is fully automated |
| G-6 | Measurable improvement | Quantitative metrics demonstrate improvement in agent-assisted task quality |

### Out of Scope

- Modifying application source code
- Changing build configurations or CI/CD pipelines
- Generating inline completion-focused configurations (focus is Agent Mode)
- Managing Copilot licensing or seat allocation
- Supporting code editors other than VS Code

---

## 3. Functional Requirements

### FR-1: Repository Discovery

The tool shall discover target repositories from Bitbucket Server 7.x using the REST API.

| Aspect | Detail |
|--------|--------|
| API endpoint | `GET /rest/api/latest/projects/{projectKey}/repos` |
| Filtering | By project key(s); optionally filter to specific repository slugs within a project |
| Pagination | Handle paginated responses (`start`, `limit`, `isLastPage`, `nextPageStart`) |
| Metadata | Retrieve repository name, slug, clone URLs (SSH and HTTPS), and default branch |
| Configuration | Target projects and optional repo filters defined in YAML configuration |

### FR-2: Repository Checkout and Branch Creation

The tool shall clone each target repository and create a dedicated feature branch.

| Aspect | Detail |
|--------|--------|
| Clone method | SSH or HTTPS (configurable); shallow clone acceptable for performance |
| Branch name | Configurable, default: `copilot-context/enhance` |
| Base branch | Configurable, default: repository's default branch (typically `main` or `master`) |
| Existing branch | If the branch already exists from a previous run, check it out and reset to base |
| Working directory | Configurable clone directory (default: system temp directory) |

### FR-3: Codebase Analysis

The tool shall analyze each cloned repository to extract structured project knowledge.

**Static analysis (deterministic, no AI required):**

| Analysis Target | What to Extract |
|----------------|-----------------|
| Build tool | Maven (`pom.xml`) or Gradle (`build.gradle`, `build.gradle.kts`); extract build/test/lint commands |
| Language and version | Java version from compiler plugin or `java.version` property |
| Framework | Spring Boot (presence of `spring-boot-starter-*`), Quarkus, Micronaut, or plain Java |
| Project structure | Source directories, test directories, resource directories, module structure (multi-module vs single) |
| Testing stack | JUnit 5, Mockito, TestContainers, Spring Boot Test, Cucumber, etc. |
| API layer | REST controllers, GraphQL, gRPC -- detected by annotations and dependencies |
| Database | JPA/Hibernate, Flyway/Liquibase migrations, connection pool (HikariCP) |
| Code quality | Checkstyle, SpotBugs, PMD, Spotless -- detected from plugin configuration |

**AI-assisted analysis (via Copilot CLI):**

| Analysis Dimension | Copilot CLI Prompt Focus |
|-------------------|--------------------------|
| Architecture and patterns | Module responsibilities, layering (controller-service-repository), design patterns used |
| Coding conventions | Naming patterns, error handling approach, logging style, API response format |
| Boundaries and constraints | What should never be modified, what should always be done a certain way |
| Common workflows | Step-by-step procedures for frequent development tasks (adding endpoints, services, tests, migrations) |

### FR-4: Copilot CLI Integration

The tool shall invoke GitHub Copilot CLI to generate project-specific context from the analyzed codebase.

| Aspect | Detail |
|--------|--------|
| Invocation | `copilot -p "<prompt>"` in programmatic (non-interactive) mode |
| Prompt strategy | Multiple focused prompts per repo (architecture, conventions, boundaries, workflows, commands) -- not one mega-prompt |
| Working directory | Copilot CLI invoked from the cloned repository root so it has access to the codebase |
| Input | Each prompt is parameterized with static analysis results (build tool, framework, structure) |
| Output parsing | Capture stdout; extract markdown content; validate structure; sanitize |
| Error handling | Retry on transient failures (network, rate limits); log and skip on persistent failures |
| Timeout | Configurable per-prompt timeout (default: 120 seconds) |

### FR-5: Agent Context File Generation

The tool shall generate files per repository in two phases. **Phase A** generates detailed technical documentation that captures deep codebase knowledge. **Phase B** generates agent context files that reference that documentation, ensuring the agent always loads project knowledge before acting.

#### FR-5.0: `documents/` (Technical Documentation -- Generated First)

The tool shall generate a `documents/` directory in each target repository containing detailed technical documentation derived from codebase analysis. These documents serve as the **persistent knowledge base** that the agent loads before every task.

| Document | Content |
|----------|---------|
| `documents/architecture.md` | System architecture, module responsibilities, layering patterns (e.g., controller-service-repository), inter-module dependencies, design patterns in use |
| `documents/coding-conventions.md` | Naming patterns, code style, error handling approach, logging standards, null-handling strategy, common idioms used in this codebase |
| `documents/api-design.md` | REST/GraphQL/gRPC patterns, URL structure, request/response formats, authentication/authorization, versioning strategy, error response format |
| `documents/testing-strategy.md` | Test frameworks, unit vs integration test patterns, fixture strategies, mocking approach, test naming conventions, CI test configuration |
| `documents/data-layer.md` | Database schema patterns, ORM usage, migration strategy (Flyway/Liquibase), repository patterns, transaction management |
| `documents/build-and-commands.md` | Build tool configuration, all build/test/lint/run commands with flags, CI/CD pipeline overview, dependency management approach |
| `documents/common-workflows.md` | Step-by-step procedures for the most frequent development tasks (adding an endpoint, adding a service, writing tests, creating migrations, etc.) |

Only generate documents relevant to what is detected in the codebase (e.g., skip `data-layer.md` if no database dependencies are found).

Each document shall:
- Be detailed enough to serve as onboarding material for a new developer
- Contain specific code examples extracted from the actual codebase (not generic examples)
- Reference actual file paths in the repository
- Be formatted as clean markdown suitable for both human reading and AI consumption

#### FR-5.1: `AGENTS.md` (Root-Level Agent Instructions -- References Documents)

The generated `AGENTS.md` shall include a **mandatory context loading section** that instructs the agent to read the generated `documents/` directory before performing any task. This ensures the agent starts every session with full project knowledge regardless of who invoked it or what prompt they wrote.

| Section | Content |
|---------|---------|
| **Mandatory context loading** | Explicit instruction to read all files in `documents/` before any task, in a specified order. Lists every generated document by path with a one-line description of what knowledge it provides |
| Persona | Role description (e.g., "You are a backend Java developer working on a Spring Boot 3.x microservice") |
| Tech stack | Language, framework, build tool, database, key libraries with versions |
| Project structure | Directory map showing where controllers, services, repositories, DTOs, tests, migrations live |
| Commands | Build (`mvn clean verify`), test (`mvn test`), lint/format (`mvn spotless:apply`), run (`mvn spring-boot:run`) -- exact commands with flags |
| Boundaries | "Never" rules (e.g., "Never modify committed Flyway migrations"), "Always" rules (e.g., "Always use MapStruct for DTO mapping") |
| Dependencies | "Ask first before adding new dependencies to pom.xml" |

#### FR-5.2: `.github/copilot-instructions.md` (Repository-Wide Coding Standards)

| Section | Content |
|---------|---------|
| Naming conventions | Class naming (`*Controller`, `*Service`, `*Repository`, `*DTO`), method naming, package naming |
| Error handling | Exception hierarchy, `@ControllerAdvice` pattern, error response format |
| Logging | SLF4J usage, log levels, structured logging fields |
| API conventions | URL structure, HTTP method usage, response envelope, pagination format |
| Security | Authentication/authorization patterns used in the project |
| Documentation | Javadoc requirements, OpenAPI annotation usage |

#### FR-5.3: `.github/instructions/*.instructions.md` (Path-Specific Rules)

Generate path-specific instruction files with `applyTo` YAML frontmatter:

| File | `applyTo` Glob | Content Focus |
|------|----------------|---------------|
| `test-conventions.instructions.md` | `src/test/**` | Test framework usage, assertion style, fixture patterns, @SpringBootTest vs unit test guidance |
| `controller-patterns.instructions.md` | `**/controller/**` | REST controller conventions, response types, validation, security annotations |
| `service-patterns.instructions.md` | `**/service/**` | Service layer conventions, transaction management, error handling |
| `repository-patterns.instructions.md` | `**/repository/**` | JPA repository patterns, custom queries, pagination |
| `migration-rules.instructions.md` | `**/db/migration/**` | Flyway/Liquibase naming conventions, immutability rules |

Only generate instruction files for patterns detected in the codebase (e.g., skip migration rules if no migration tool is detected).

#### FR-5.4: `.github/prompts/*.prompt.md` (Reusable Task Templates)

Generate prompt templates for common workflows detected in the codebase:

| Prompt File | YAML Frontmatter | Purpose |
|-------------|-------------------|---------|
| `add-rest-endpoint.prompt.md` | `agent: agent` | Scaffold a new REST endpoint with controller, service, repository, DTO, mapper, migration, and test |
| `write-integration-tests.prompt.md` | `agent: agent` | Write integration tests for a class following project patterns |
| `add-service-class.prompt.md` | `agent: agent` | Create a new service with proper annotations, transaction management, and error handling |
| `database-migration.prompt.md` | `agent: agent` | Create a Flyway/Liquibase migration with corresponding entity and repository updates |
| `add-dto-mapping.prompt.md` | `agent: agent` | Create DTOs and MapStruct mapper interfaces |

Each prompt template shall include:
- Variable placeholders using `${input:variableName}` syntax
- References to existing pattern files using `#file:` syntax where applicable
- Step-by-step instructions the agent should follow
- Validation steps (build, test)

Only generate prompt templates for workflows relevant to the detected tech stack.

#### FR-5.5: `.github/agents/*.agent.md` (Custom Agent Profiles)

Generate specialized agent profiles:

| Agent File | Role | Tools |
|------------|------|-------|
| `test-specialist.agent.md` | Write and fix tests following project patterns | `read`, `edit`, `search`, `execute` |
| `api-reviewer.agent.md` | Review API endpoints for consistency and security | `read`, `search` |
| `migration-helper.agent.md` | Database schema changes, migration creation, entity updates | `read`, `edit`, `search`, `execute` |

Each agent shall include:
- `name` and `description` in YAML frontmatter
- `tools` list appropriate for the role
- Detailed persona instructions in the markdown body
- Explicit boundaries for the agent's scope

Only generate agents relevant to the detected tech stack.

#### FR-5.6: `.github/skills/**/SKILL.md` (Workflow Runbooks)

Generate multi-step workflow skills for complex operations:

| Skill Directory | Purpose |
|----------------|---------|
| `database-migration/SKILL.md` | Step-by-step: create migration, update entity, update repository, run migration, run tests |
| `new-endpoint/SKILL.md` | Full endpoint creation workflow from controller to integration test |
| `refactor-service/SKILL.md` | Safe refactoring workflow with test verification at each step |

Each skill shall include:
- `name` and `description` in YAML frontmatter
- Numbered step-by-step instructions
- Verification commands to run after each step
- Rollback guidance if a step fails

#### FR-5.7: `.vscode/settings.json` (Agent Mode Settings)

| Setting | Value | Purpose |
|---------|-------|---------|
| `github.copilot.chat.codeGeneration.useInstructionFiles` | `true` | Enable instruction file consumption |
| `chat.agentSkillsLocations` | `{".github/skills/**": true}` | Register skills directory |
| `chat.agentFilesLocations` | `{".github/agents/**": true}` | Register custom agents directory |
| `chat.promptFiles` | `true` | Enable prompt file discovery |
| `github.copilot.enable` | `{"*": true}` | Enable Copilot for all file types |

If a `.vscode/settings.json` already exists, merge the new settings without overwriting existing entries.

#### FR-5.8: `.vscode/extensions.json` (Recommended Extensions)

Recommend GitHub Copilot and Copilot Chat extensions. If the file already exists, merge recommendations without duplicating.

### FR-6: Commit, Push, and Pull Request Creation

| Aspect | Detail |
|--------|--------|
| Commit message | Descriptive, e.g., "Add Copilot Agent Mode context files (AGENTS.md, instructions, prompts, agents, skills)" |
| Push | Push the feature branch to the remote |
| PR creation | Via Bitbucket Server REST API (`POST /rest/api/latest/projects/{key}/repos/{slug}/pull-requests`) |
| PR title | Configurable template, default: `[Copilot Agent] Add AI agent context files for {repo_name}` |
| PR description | Auto-generated summary listing all created files by category with brief descriptions |
| Reviewers | Configurable list of default reviewers |
| PR target | The repository's default branch |

### FR-7: Execution Report

The tool shall generate a report after processing all repositories.

| Report Content | Detail |
|----------------|--------|
| Summary | Total repos targeted, processed, succeeded, failed, skipped |
| Per-repo detail | Repository name, branch created, files generated (by category), PR URL, errors (if any) |
| File inventory | Total files generated across all repos, broken down by type (AGENTS.md, instructions, prompts, agents, skills, vscode) |
| Format | JSON (machine-readable) + human-readable markdown summary |
| Output location | Configurable; default: `./reports/` directory |

---

## 4. Non-Functional Requirements

### NFR-1: Scalability

| Aspect | Requirement |
|--------|-------------|
| Pilot | Support processing fewer than 20 repositories in a single run |
| Scale target | Designed to scale to 500+ repositories |
| Concurrency | Configurable thread pool (default: 4 concurrent repos) |
| Rate limiting | Respect Bitbucket Server API rate limits; configurable delay between API calls |

### NFR-2: Idempotency

| Aspect | Requirement |
|--------|-------------|
| Re-runs | Running the tool against a previously processed repository shall update existing files, not create duplicates |
| File merging | `.vscode/settings.json` and `.vscode/extensions.json` shall be merged with existing content |
| Markdown files | `AGENTS.md`, instruction files, prompts, agents, and skills shall be regenerated (overwrite) unless a merge strategy is configured |
| Branch handling | If the feature branch already exists, reset to the base branch before regenerating |

### NFR-3: Configurability

All behavior shall be configurable via a YAML configuration file:

| Configuration Area | Options |
|-------------------|---------|
| Bitbucket connection | Base URL, auth type (token/basic), credentials via environment variables |
| Target repositories | Project keys, optional repo slug filters |
| Branch settings | Branch name, base branch |
| Processing | Concurrency level, clone directory, cleanup policy |
| File generation | Enable/disable each file category independently |
| PR settings | Title template, description template, default reviewers |
| Copilot CLI | Path to CLI binary, per-prompt timeout, retry count |

### NFR-4: Security

| Aspect | Requirement |
|--------|-------------|
| Credentials | Never stored in configuration files; sourced from environment variables or a secrets manager |
| Clone cleanup | Working directories cleaned up after processing (configurable) |
| Generated content | No secrets, passwords, or sensitive configuration values in generated files |
| Network | Support HTTPS for Bitbucket API calls; support SSH for git clone |

### NFR-5: Resilience

| Aspect | Requirement |
|--------|-------------|
| Per-repo isolation | Failure processing one repository shall not block processing of others |
| Retry logic | Transient failures (network, API rate limits) shall be retried with exponential backoff |
| Partial completion | If a repo fails mid-processing, already-cloned data is cleaned up |
| Copilot CLI failures | If a Copilot CLI prompt fails, proceed with remaining prompts; use static analysis data for the failed dimension |

### NFR-6: Observability

| Aspect | Requirement |
|--------|-------------|
| Logging | Structured logging with per-repo context (repo name, project key) |
| Progress | Real-time progress indicator showing repos processed / total |
| Log levels | Configurable (DEBUG, INFO, WARN, ERROR); default: INFO |
| Audit trail | Log every API call, git operation, Copilot CLI invocation, and file generation event |

---

## 5. Assumptions and Constraints

### Assumptions

| ID | Assumption |
|----|-----------|
| A-1 | Bitbucket Server 7.x is accessible via REST API from the machine running the tool |
| A-2 | GitHub Copilot CLI is installed, authenticated, and functional on the runner machine |
| A-3 | The runner machine has SSH or HTTPS clone access to all target repositories |
| A-4 | Target repositories are primarily Java applications using Maven or Gradle |
| A-5 | Repositories have a standard branch (main/master/develop) that serves as the PR target |
| A-6 | The Bitbucket user account used by the tool has permission to create branches and pull requests |
| A-7 | Generated context files will be reviewed by the development team before merging |

### Constraints

| ID | Constraint |
|----|-----------|
| C-1 | The tool must not modify any application source code -- only add context/configuration files |
| C-2 | The tool must not modify build configurations (pom.xml, build.gradle) |
| C-3 | All generated files must be delivered via pull requests, never pushed directly to the default branch |
| C-4 | Copilot CLI output quality depends on the model version available; generated content should be treated as a starting point for human review |
| C-5 | The tool must run on a machine with internet access (for Copilot CLI) and network access to the Bitbucket Server instance |

---

## 6. Metrics and Measurement

### Agent Mode Performance Metrics

| Metric | Definition | Measurement Method | Target |
|--------|-----------|-------------------|--------|
| Agent task success rate | % of agent-mode tasks that produce merge-ready code without significant rework | Track PR approval rate for agent-generated changes, before vs after | 30-50% improvement |
| Code review rejection rate | % of agent-generated PRs requiring major rework (not just style nits) | Code review data from Bitbucket, before vs after | 40-60% reduction |
| Task completion time | Elapsed time from prompt to working, tested code for standardized tasks | Developer self-reporting or IDE telemetry | 25-40% faster |
| Prompt template adoption | % of agent interactions using generated templates vs ad-hoc prompts | Survey + optional telemetry | >60% within 3 months |
| Custom agent usage | Frequency and distribution of custom agent selection | Survey | >40% of agent sessions use a custom agent |
| Developer satisfaction | Self-reported satisfaction with agent mode, segmented by experience level | Survey (1-5 scale), before vs after | 1+ point improvement across all levels |

### Measurement Process

1. **Baseline (Week 0-2):** Collect metrics for 2 weeks before enabling context files on pilot repos. Survey all developers on the pilot teams.
2. **Enablement (Week 2):** Merge the generated PRs on pilot repos. Conduct a brief training session on using prompt templates and custom agents.
3. **Post-measurement (Week 2-6):** Collect the same metrics for 4 weeks after enablement.
4. **Analysis (Week 6-7):** Compare before/after metrics. Segment by developer experience level to validate the leveling effect. Present findings to management.
5. **Iteration (Week 7-8):** Refine prompt quality, add/modify prompt templates based on feedback. Re-run the tool to update context files.
6. **Scale decision (Week 8+):** Based on pilot results, decide whether to roll out to additional projects.

---

## 7. Glossary

| Term | Definition |
|------|-----------|
| **Agent Mode** | GitHub Copilot's autonomous mode that reads, edits, and executes across multiple files to complete multi-step coding tasks |
| **AGENTS.md** | A markdown file at the repository root that provides agent persona, commands, project structure, and boundaries |
| **Copilot CLI** | The command-line interface for GitHub Copilot, usable programmatically via `copilot -p "<prompt>"` |
| **Prompt template** | A `.prompt.md` file that defines a reusable, parameterized task workflow for the agent |
| **Custom agent** | An `.agent.md` file that defines a specialized agent persona with specific tools and boundaries |
| **Skill** | A `SKILL.md` file that defines a multi-step workflow runbook the agent can follow |
| **Path-specific instruction** | An `.instructions.md` file with an `applyTo` glob that activates when the agent edits matching files |
| **Bitbucket Server** | Atlassian's self-hosted Git repository management product (version 7.x in this context) |
| **Idempotent** | A property of the tool: running it multiple times on the same repo produces the same result without duplication |
