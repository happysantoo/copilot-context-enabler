<!--
Copyright 2026 Copilot Context Enabler Contributors
SPDX-License-Identifier: Apache-2.0
-->

# Implementation Plan: Copilot Context Enabler (Python)

**Version:** 1.0
**Language:** Python 3.10+
**Estimated Timeline:** 6-7 weeks (Phases 1-4) + 1-2 weeks pilot (Phase 5)
**Team Size:** 1-2 developers

---

## 1. Technology Stack

### Runtime and Build

| Component | Technology | Version |
|-----------|-----------|---------|
| Language | Python | 3.10+ |
| Package manager | pip / uv | Latest |
| Project config | `pyproject.toml` (PEP 621) | -- |
| Virtual environment | venv or uv | -- |

### Dependencies

| Library | Purpose | Version |
|---------|---------|---------|
| `requests` | HTTP client for Bitbucket Server REST API | 2.31+ |
| `gitpython` | Git operations (clone, branch, commit, push) | 3.1+ |
| `pyyaml` | YAML configuration parsing | 6.0+ |
| `jinja2` | Template engine for prompts and output files | 3.1+ |
| `rich` | Console output, progress bars, structured logging | 13.0+ |
| `click` | CLI framework for entry point | 8.1+ |
| `pydantic` | Configuration and data model validation | 2.5+ |

### Dev Dependencies

| Library | Purpose |
|---------|---------|
| `pytest` | Unit and integration testing |
| `pytest-cov` | Code coverage reporting |
| `responses` | HTTP request mocking for Bitbucket API tests |
| `ruff` | Linting and formatting |
| `mypy` | Static type checking |

### External Tools

| Tool | Purpose |
|------|---------|
| GitHub Copilot CLI | AI-assisted codebase analysis (invoked via `subprocess`) |
| Git | Version control operations (used via GitPython) |

---

## 2. Project Structure

```
copilot-context-enabler/
├── pyproject.toml
├── README.md
├── config.example.yaml
├── src/
│   └── copilot_enabler/
│       ├── __init__.py
│       ├── cli.py                          # Click-based CLI entry point
│       ├── config.py                       # ConfigLoader (Pydantic models)
│       ├── models.py                       # Shared data models (AnalysisResult, RepoResult, etc.)
│       │
│       ├── bitbucket/
│       │   ├── __init__.py
│       │   └── client.py                   # BitbucketAPIClient
│       │
│       ├── git/
│       │   ├── __init__.py
│       │   └── operations.py               # GitOperations
│       │
│       ├── analysis/
│       │   ├── __init__.py
│       │   ├── static_analyzer.py          # StaticAnalyzer
│       │   ├── copilot_bridge.py           # CopilotCLIBridge
│       │   └── prompt_engine.py            # PromptTemplateEngine
│       │
│       ├── generators/
│       │   ├── __init__.py
│       │   ├── base.py                     # BaseGenerator abstract class
│       │   ├── agents_md.py                # AgentsMdGenerator
│       │   ├── instructions.py             # InstructionGenerator
│       │   ├── documentation.py             # Phase A: documents/ generator
│       │   ├── prompt_files.py             # Phase B: PromptFileGenerator
│       │   ├── custom_agents.py            # Phase B: CustomAgentGenerator
│       │   ├── skills.py                   # Phase B: SkillGenerator
│       │   └── vscode_config.py            # Phase B: VSCodeConfigGenerator
│       │
│       ├── templates/
│       │   ├── prompts/                    # Jinja2 templates for Copilot CLI prompts
│       │   │   ├── architecture.md.j2
│       │   │   ├── conventions.md.j2
│       │   │   ├── boundaries.md.j2
│       │   │   ├── workflows.md.j2
│       │   │   └── commands.md.j2
│       │   └── output/                     # Jinja2 templates for generated files
│       │       ├── docs/                   # Templates for documents/ (Phase A)
│       │       │   ├── architecture.md.j2
│       │       │   ├── coding_conventions.md.j2
│       │       │   ├── api_design.md.j2
│       │       │   ├── testing_strategy.md.j2
│       │       │   ├── data_layer.md.j2
│       │       │   ├── build_and_commands.md.j2
│       │       │   └── common_workflows.md.j2
│       │       ├── agents_md.md.j2         # Template includes mandatory doc loading
│       │       ├── copilot_instructions.md.j2
│       │       ├── path_instruction.md.j2
│       │       ├── prompt_template.md.j2
│       │       ├── custom_agent.md.j2
│       │       ├── skill.md.j2
│       │       ├── vscode_settings.json.j2
│       │       └── vscode_extensions.json.j2
│       │
│       ├── orchestrator.py                 # RepoProcessor (main loop)
│       └── reporter.py                     # ReportGenerator
│
├── tests/
│   ├── conftest.py                         # Shared fixtures
│   ├── test_config.py
│   ├── test_bitbucket_client.py
│   ├── test_git_operations.py
│   ├── test_static_analyzer.py
│   ├── test_copilot_bridge.py
│   ├── test_generators/
│   │   ├── test_documentation.py
│   │   ├── test_agents_md.py
│   │   ├── test_instructions.py
│   │   ├── test_prompt_files.py
│   │   ├── test_custom_agents.py
│   │   ├── test_skills.py
│   │   └── test_vscode_config.py
│   ├── test_orchestrator.py
│   └── fixtures/                           # Sample repos, config files, API responses
│       ├── sample_pom.xml
│       ├── sample_build.gradle
│       ├── sample_config.yaml
│       └── sample_api_responses/
│
└── reports/                                # Generated execution reports (gitignored)
```

---

## 3. Phased Implementation Plan

### Phase 1: Foundation (Week 1-2)

**Goal:** Establish the project skeleton, configuration system, Bitbucket API client, and Git operations.

#### Week 1: Project Setup and Bitbucket Client

| Task | Detail | Output |
|------|--------|--------|
| 1.1 Initialize project | Create `pyproject.toml` with all dependencies, `README.md`, `config.example.yaml` | Buildable project skeleton |
| 1.2 Configuration system | `config.py` with Pydantic models for all config sections; YAML loading; env var resolution | `ConfigLoader` class |
| 1.3 Data models | `models.py` with `AnalysisResult`, `RepoResult`, `GeneratedFile`, `RunReport` dataclasses | Shared type definitions |
| 1.4 Bitbucket API client | `bitbucket/client.py` with methods: `list_repos()`, `get_repo()`, `get_default_branch()`, `create_branch()`, `create_pull_request()` | `BitbucketAPIClient` class |
| 1.5 Pagination handling | Generic paginator for Bitbucket Server's `start`/`limit`/`isLastPage` pattern | Integrated into client |
| 1.6 Unit tests | Tests for config loading (valid, invalid, missing env vars) and Bitbucket client (mocked HTTP) | `test_config.py`, `test_bitbucket_client.py` |

#### Week 2: Git Operations and CLI Entry Point

| Task | Detail | Output |
|------|--------|--------|
| 2.1 Git operations | `git/operations.py` with methods: `clone()`, `create_branch()`, `add_all()`, `commit()`, `push()`, `cleanup()` | `GitOperations` class |
| 2.2 Shallow clone support | Optional `--depth 1` for performance; configurable | Integrated into clone |
| 2.3 Branch handling | Create branch from base; handle existing branch (checkout + reset) | Idempotent branching |
| 2.4 CLI entry point | `cli.py` with Click: `run` command (config file path, dry-run flag, verbosity) | `copilot-enabler run --config config.yaml` |
| 2.5 Logging setup | Structured logging via `rich` + Python `logging`; per-repo context | Log configuration |
| 2.6 Unit tests | Tests for git operations (using temp directories with git init) | `test_git_operations.py` |

**Phase 1 deliverable:** The tool can read config, connect to Bitbucket, list repos, clone them, create branches, and push -- with no analysis or file generation yet.

---

### Phase 2: Codebase Analysis Engine (Week 2-4)

**Goal:** Build the static analyzer and Copilot CLI integration to extract project knowledge.

#### Week 3: Static Analyzer

| Task | Detail | Output |
|------|--------|--------|
| 3.1 Build tool detection | Detect Maven (`pom.xml`), Gradle (`build.gradle`, `build.gradle.kts`); extract artifact info | Build tool identification |
| 3.2 Maven POM parsing | Parse `pom.xml` with `xml.etree.ElementTree`: extract dependencies, plugins, properties, modules | Dependency/plugin extraction |
| 3.3 Framework detection | Detect Spring Boot, Quarkus, Micronaut from dependencies | Framework identification |
| 3.4 Command extraction | Derive build/test/lint/run commands from build tool and detected plugins | Command list |
| 3.5 Project structure mapping | Walk the directory tree; identify source, test, resource, migration directories | Directory tree model |
| 3.6 Pattern detection | Identify controller-service-repository pattern, DTO/mapper usage, test frameworks, migration tools | Pattern list |
| 3.7 Unit tests | Tests against fixture pom.xml and build.gradle files | `test_static_analyzer.py` |

#### Week 4: Copilot CLI Bridge and Prompt Engine

| Task | Detail | Output |
|------|--------|--------|
| 4.1 Prompt templates | Create 5 Jinja2 templates for Copilot CLI prompts (architecture, conventions, boundaries, workflows, commands) | Template files in `templates/prompts/` |
| 4.2 PromptTemplateEngine | Render Jinja2 templates with `AnalysisResult` data | `PromptTemplateEngine` class |
| 4.3 CopilotCLIBridge | Invoke `copilot -p` via `subprocess.run()` with timeout, capture stdout | `CopilotCLIBridge` class |
| 4.4 Response parsing | Extract markdown sections from Copilot CLI output; validate structure | Parser functions |
| 4.5 Retry logic | Retry on transient failures (subprocess timeout, non-zero exit code) with configurable backoff | Integrated into bridge |
| 4.6 Graceful degradation | If a prompt fails, return empty result for that dimension; log warning | Partial result handling |
| 4.7 Integration test | End-to-end test: static analysis -> prompt rendering -> CLI invocation (requires Copilot CLI) | Manual integration test |

**Phase 2 deliverable:** Given a cloned repo, the tool can produce a full `AnalysisResult` and AI-generated content for all dimensions.

---

### Phase 3: Documentation and Agent File Generation (Week 4-6)

**Goal:** Build the two-phase generation pipeline: first generate technical documentation (`documents/`), then generate agent files that reference those documents.

#### Week 5: Documentation Generator (Phase A) and Core Agent Generators (Phase B)

| Task | Detail | Output |
|------|--------|--------|
| 5.1 BaseGenerator | Abstract base class with common interface: `generate(analysis, ai_content, repo_path, generated_docs) -> list[GeneratedFile]` | `generators/base.py` |
| 5.2 DocumentationGenerator | Generate `documents/` directory with detailed technical docs: `architecture.md`, `coding-conventions.md`, `api-design.md`, `testing-strategy.md`, `data-layer.md`, `build-and-commands.md`, `common-workflows.md`. Conditional: only create docs for detected patterns. Each doc contains specific code examples and real file paths from the repo | `generators/documentation.py` |
| 5.3 Documentation output templates | Jinja2 templates for each document type in `templates/output/docs/` | `templates/output/docs/*.j2` |
| 5.4 AgentsMdGenerator | Generate `AGENTS.md` with **mandatory context loading section** that lists all generated `documents/` files by path, instructing the agent to read them before any task. Combines static commands + AI persona/boundaries | `generators/agents_md.py` |
| 5.5 InstructionGenerator | Generate `copilot-instructions.md` + conditional path-specific files based on detected patterns | `generators/instructions.py` |
| 5.6 VSCodeConfigGenerator | Generate/merge `.vscode/settings.json` and `extensions.json` | `generators/vscode_config.py` |
| 5.7 Unit tests | Test DocumentationGenerator (conditional doc creation, content structure); test AgentsMdGenerator (verify mandatory loading section lists generated docs) | `test_generators/` |

#### Week 6: Advanced Generators (Prompts, Agents, Skills) and Idempotency

| Task | Detail | Output |
|------|--------|--------|
| 6.1 PromptFileGenerator | Generate `.prompt.md` files for detected workflows; include YAML frontmatter and `${input:*}` variables. Each prompt references relevant generated docs (e.g., `add-rest-endpoint.prompt.md` references `documents/api-design.md` and `documents/common-workflows.md`) | `generators/prompt_files.py` |
| 6.2 CustomAgentGenerator | Generate `.agent.md` files for test-specialist, api-reviewer, migration-helper (conditional). Each agent's instructions include directives to read relevant docs (e.g., `test-specialist` reads `documents/testing-strategy.md`) | `generators/custom_agents.py` |
| 6.3 SkillGenerator | Generate `SKILL.md` files in subdirectories for multi-step workflows (conditional). Skills reference relevant docs for context | `generators/skills.py` |
| 6.4 Idempotency logic | For JSON files: deep merge. For markdown: overwrite (with option to merge sections). Detect existing files before writing | Integrated into generators |
| 6.5 Unit tests | Test conditional generation (e.g., skip migration files when no Flyway detected) | `test_generators/` |

**Phase 3 deliverable:** Given analysis data and AI content, the tool produces all agent context files correctly.

---

### Phase 4: Orchestration and PR Creation (Week 6-7)

**Goal:** Wire everything together into the main processing loop with concurrency, PR creation, and reporting.

#### Week 7: Orchestrator, PR, and Report

| Task | Detail | Output |
|------|--------|--------|
| 7.1 RepoProcessor | Main orchestration: config -> discover -> for each repo (clone -> branch -> analyze -> generate -> commit -> push -> PR) | `orchestrator.py` |
| 7.2 Thread pool | `concurrent.futures.ThreadPoolExecutor` with configurable size; per-repo error isolation | Concurrent processing |
| 7.3 PR creation | Invoke `BitbucketAPIClient.create_pull_request()` with auto-generated description listing all files by category | PR creation |
| 7.4 PR description builder | Generate markdown description from list of `GeneratedFile` objects, organized by category | Description template |
| 7.5 ReportGenerator | Aggregate `RepoResult` objects into JSON and Markdown reports | `reporter.py` |
| 7.6 Dry-run mode | `--dry-run` flag: run analysis and generation but skip commit/push/PR creation; print what would be done | CLI flag |
| 7.7 End-to-end test | Full pipeline test against a test Bitbucket project (or mocked) | Integration test |
| 7.8 Error handling | Per-repo try/catch with logging; ensure cleanup on failure; aggregate errors in report | Resilient processing |

**Phase 4 deliverable:** The tool runs end-to-end: reads config, discovers repos, processes each with concurrency, creates PRs, and generates a report.

---

### Phase 5: Pilot and Measurement (Week 7-8)

**Goal:** Run against pilot repositories, collect metrics, iterate on quality.

| Task | Detail | Timeline |
|------|--------|----------|
| 8.1 Baseline collection | Record current agent task success rate, code review rejection rate, developer satisfaction (survey) | Week 7 |
| 8.2 Pilot run | Execute against < 20 pilot repositories; review generated PRs with teams | Week 7-8 |
| 8.3 Prompt iteration | Based on PR review feedback, refine Copilot CLI prompt templates for better output | Week 8 |
| 8.4 Template iteration | Refine output templates (AGENTS.md structure, prompt templates, agent definitions) based on developer feedback | Week 8 |
| 8.5 Re-run | Execute again after refinements; verify improved output | Week 8 |
| 8.6 Post-measurement | Collect the same metrics after 2-4 weeks of context file usage | Week 10-12 |
| 8.7 Scale-up plan | Document findings, prepare recommendation for scaling to additional projects | Week 12 |

---

## 4. Key Implementation Details

### Configuration Loading (Pydantic)

```python
from pydantic import BaseModel, Field
from typing import Optional

class BitbucketConfig(BaseModel):
    base_url: str
    auth_type: str = "token"

class TargetConfig(BaseModel):
    project_key: str
    repos: list[str] = Field(default_factory=list)

class BranchConfig(BaseModel):
    name: str = "copilot-context/enhance"
    base: str = "main"

class ProcessingConfig(BaseModel):
    concurrency: int = 4
    clone_dir: str = "/tmp/copilot-enabler"
    clone_depth: int = 1
    cleanup_after: bool = True

class GenerationConfig(BaseModel):
    agents_md: bool = True
    copilot_instructions: bool = True
    path_instructions: bool = True
    prompt_templates: bool = True
    custom_agents: bool = True
    skills: bool = True
    vscode_config: bool = True

class AppConfig(BaseModel):
    bitbucket: BitbucketConfig
    targets: list[TargetConfig]
    branch: BranchConfig = BranchConfig()
    processing: ProcessingConfig = ProcessingConfig()
    generation: GenerationConfig = GenerationConfig()
```

### Copilot CLI Invocation

```python
import subprocess
from pathlib import Path

class CopilotCLIBridge:
    def invoke(self, prompt: str, working_dir: Path, timeout: int = 120) -> str:
        result = subprocess.run(
            ["copilot", "-p", prompt],
            cwd=working_dir,
            capture_output=True,
            text=True,
            timeout=timeout,
        )
        if result.returncode != 0:
            raise CopilotCLIError(result.stderr)
        return result.stdout
```

### Generator Interface

```python
from abc import ABC, abstractmethod
from pathlib import Path

class BaseGenerator(ABC):
    @abstractmethod
    def generate(
        self,
        analysis: AnalysisResult,
        ai_content: dict[str, str],
        repo_path: Path,
    ) -> list[GeneratedFile]:
        ...

    def should_generate(self, analysis: AnalysisResult) -> bool:
        return True
```

All generators implement `should_generate()` to conditionally skip file creation when preconditions are not met (e.g., `SkillGenerator.should_generate()` returns `False` if no migration tool or REST API is detected).

### Thread Pool Orchestration

```python
from concurrent.futures import ThreadPoolExecutor, as_completed

class RepoProcessor:
    def process_all(self, config: AppConfig) -> list[RepoResult]:
        repos = self._discover_repos(config)
        results = []

        with ThreadPoolExecutor(max_workers=config.processing.concurrency) as pool:
            futures = {
                pool.submit(self._process_repo, repo, config): repo
                for repo in repos
            }
            for future in as_completed(futures):
                repo = futures[future]
                try:
                    results.append(future.result())
                except Exception as e:
                    results.append(RepoResult(
                        repo=repo, status="failed", error=str(e)
                    ))

        return results
```

---

## 5. CLI Usage

```bash
# Install
pip install -e .

# Run with config
copilot-enabler run --config config.yaml

# Dry run (no commits, pushes, or PRs)
copilot-enabler run --config config.yaml --dry-run

# Verbose output
copilot-enabler run --config config.yaml --verbose

# Process a single repo (override targets)
copilot-enabler run --config config.yaml --project PROJ1 --repo order-service
```

---

## 6. Risk Mitigation

| Risk | Mitigation |
|------|------------|
| Copilot CLI output quality varies | Multiple focused prompts; validate output structure; human review via PR |
| Copilot CLI rate limiting | Configurable delay between invocations; retry with backoff |
| Large repos slow to clone | Shallow clone (`--depth 1`); skip repos above configurable size threshold |
| Generated files conflict with existing content | Merge logic for JSON; overwrite for markdown (with backup option) |
| Team unfamiliar with Python | Comprehensive README; well-structured code; type hints throughout; CI/CD pipeline |

---

## 7. Definition of Done

Each phase is complete when:

| Phase | Criteria |
|-------|----------|
| Phase 1 | Config loads, Bitbucket client lists repos, Git operations clone/branch/commit/push work; >80% test coverage |
| Phase 2 | StaticAnalyzer produces correct AnalysisResult for Maven and Gradle projects; CopilotCLIBridge invokes CLI and parses output; >80% test coverage |
| Phase 3 | All 6 generators produce valid output files; conditional generation works; idempotency verified; >80% test coverage |
| Phase 4 | End-to-end run completes: config -> discover -> process (concurrent) -> PRs created -> report generated; dry-run mode works |
| Phase 5 | Pilot completed on <20 repos; baseline metrics collected; at least one iteration of prompt refinement done; findings documented |
