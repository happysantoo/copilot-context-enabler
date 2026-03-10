<!--
Copyright 2026 Copilot Context Enabler Contributors
SPDX-License-Identifier: Apache-2.0
-->

# Implementation Plan: Copilot Context Enabler (Java)

**Version:** 1.0
**Language:** Java 17+ (Java 21 recommended for Virtual Threads)
**Build Tool:** Maven
**Estimated Timeline:** 7-9 weeks (Phases 1-4) + 1-2 weeks pilot (Phase 5)
**Team Size:** 1-2 developers

---

## 1. Technology Stack

### Runtime and Build

| Component | Technology | Version |
|-----------|-----------|---------|
| Language | Java | 17+ (21 recommended) |
| Build tool | Maven | 3.9+ |
| Packaging | Executable JAR (maven-shade-plugin or Spring Boot plugin) | -- |

### Dependencies

| Library | Group ID | Purpose | Version |
|---------|----------|---------|---------|
| OkHttp | `com.squareup.okhttp3` | HTTP client for Bitbucket Server REST API | 4.12+ |
| JGit | `org.eclipse.jgit` | Git operations (clone, branch, commit, push) | 6.8+ |
| SnakeYAML | `org.yaml` | YAML configuration parsing | 2.2+ |
| Freemarker | `org.freemarker` | Template engine for prompts and output files | 2.3.32+ |
| Jackson | `com.fasterxml.jackson.core` | JSON processing (reports, settings.json merge) | 2.16+ |
| Jackson YAML | `com.fasterxml.jackson.dataformat` | YAML deserialization to typed POJOs | 2.16+ |
| SLF4J | `org.slf4j` | Logging facade | 2.0+ |
| Logback | `ch.qos.logback` | Logging implementation | 1.4+ |
| Picocli | `info.picocli` | CLI framework for entry point | 4.7+ |
| Maven Model | `org.apache.maven` | Native POM XML parsing | 3.9+ |

### Test Dependencies

| Library | Purpose | Version |
|---------|---------|---------|
| JUnit 5 | Unit and integration testing | 5.10+ |
| Mockito | Mocking framework | 5.8+ |
| WireMock | HTTP API mocking for Bitbucket client tests | 3.3+ |
| AssertJ | Fluent assertions | 3.25+ |

### External Tools

| Tool | Purpose |
|------|---------|
| GitHub Copilot CLI | AI-assisted codebase analysis (invoked via `ProcessBuilder`) |
| Git | Version control operations (used via JGit) |

---

## 2. Project Structure

```
copilot-context-enabler/
├── pom.xml
├── README.md
├── config.example.yaml
│
├── src/
│   ├── main/
│   │   ├── java/
│   │   │   └── com/company/copilotenabler/
│   │   │       ├── CopilotEnablerApp.java                  # Main class + Picocli command
│   │   │       │
│   │   │       ├── config/
│   │   │       │   ├── AppConfig.java                       # Root config POJO
│   │   │       │   ├── BitbucketConfig.java
│   │   │       │   ├── TargetConfig.java
│   │   │       │   ├── BranchConfig.java
│   │   │       │   ├── ProcessingConfig.java
│   │   │       │   ├── GenerationConfig.java
│   │   │       │   ├── CopilotCliConfig.java
│   │   │       │   ├── PrConfig.java
│   │   │       │   └── ConfigLoader.java                    # YAML loading + env var resolution
│   │   │       │
│   │   │       ├── model/
│   │   │       │   ├── AnalysisResult.java                  # Static + AI analysis output
│   │   │       │   ├── AiContent.java                       # Parsed Copilot CLI responses
│   │   │       │   ├── RepoResult.java                      # Per-repo processing result
│   │   │       │   ├── GeneratedFile.java                   # File path + category metadata
│   │   │       │   └── RunReport.java                       # Aggregated run report
│   │   │       │
│   │   │       ├── bitbucket/
│   │   │       │   ├── BitbucketApiClient.java              # REST API client
│   │   │       │   ├── BitbucketPaginator.java              # Generic paginated response handler
│   │   │       │   └── dto/                                 # API response DTOs
│   │   │       │       ├── RepoDto.java
│   │   │       │       ├── BranchDto.java
│   │   │       │       ├── PullRequestDto.java
│   │   │       │       └── PagedResponse.java
│   │   │       │
│   │   │       ├── git/
│   │   │       │   └── GitOperations.java                   # JGit-based git operations
│   │   │       │
│   │   │       ├── analysis/
│   │   │       │   ├── StaticAnalyzer.java                  # Deterministic codebase analysis
│   │   │       │   ├── MavenAnalyzer.java                   # POM parsing via maven-model
│   │   │       │   ├── GradleAnalyzer.java                  # Gradle build file parsing
│   │   │       │   ├── CopilotCliBridge.java                # ProcessBuilder-based CLI invocation
│   │   │       │   └── PromptTemplateEngine.java            # Freemarker prompt rendering
│   │   │       │
│   │   │       ├── generators/
│   │   │       │   ├── FileGenerator.java                   # Interface for all generators
│   │   │       │   ├── GeneratorRegistry.java               # Runs Phase A then Phase B
│   │   │       │   ├── DocumentationGenerator.java          # Phase A: documents/ generator
│   │   │       │   ├── AgentsMdGenerator.java               # Phase B: AGENTS.md (mandatory doc loading)
│   │   │       │   ├── InstructionGenerator.java            # Phase B: copilot-instructions + path-specific
│   │   │       │   ├── PromptFileGenerator.java             # Phase B: .prompt.md (references docs)
│   │   │       │   ├── CustomAgentGenerator.java            # Phase B: .agent.md (references docs)
│   │   │       │   ├── SkillGenerator.java                  # Phase B: SKILL.md (references docs)
│   │   │       │   └── VSCodeConfigGenerator.java           # Phase B: .vscode/ files
│   │   │       │
│   │   │       ├── orchestrator/
│   │   │       │   └── RepoProcessor.java                   # Main processing loop
│   │   │       │
│   │   │       └── report/
│   │   │           └── ReportGenerator.java                 # JSON + Markdown report output
│   │   │
│   │   └── resources/
│   │       ├── templates/
│   │       │   ├── prompts/                                 # Freemarker prompt templates
│   │       │   │   ├── architecture.ftl
│   │       │   │   ├── conventions.ftl
│   │       │   │   ├── boundaries.ftl
│   │       │   │   ├── workflows.ftl
│   │       │   │   └── commands.ftl
│   │       │   └── output/                                  # Freemarker output file templates
│   │       │       ├── docs/                                # Templates for documents/ (Phase A)
│   │       │       │   ├── architecture.ftl
│   │       │       │   ├── coding_conventions.ftl
│   │       │       │   ├── api_design.ftl
│   │       │       │   ├── testing_strategy.ftl
│   │       │       │   ├── data_layer.ftl
│   │       │       │   ├── build_and_commands.ftl
│   │       │       │   └── common_workflows.ftl
│   │       │       ├── agents_md.ftl                        # Includes mandatory doc loading
│   │       │       ├── copilot_instructions.ftl
│   │       │       ├── path_instruction.ftl
│   │       │       ├── prompt_template.ftl
│   │       │       ├── custom_agent.ftl
│   │       │       ├── skill.ftl
│   │       │       ├── vscode_settings.json.ftl
│   │       │       └── vscode_extensions.json.ftl
│   │       └── logback.xml                                  # Logging configuration
│   │
│   └── test/
│       ├── java/
│       │   └── com/company/copilotenabler/
│       │       ├── config/
│       │       │   └── ConfigLoaderTest.java
│       │       ├── bitbucket/
│       │       │   └── BitbucketApiClientTest.java          # WireMock-based
│       │       ├── git/
│       │       │   └── GitOperationsTest.java               # Temp dir + JGit
│       │       ├── analysis/
│       │       │   ├── StaticAnalyzerTest.java
│       │       │   ├── MavenAnalyzerTest.java
│       │       │   └── CopilotCliBridgeTest.java
│       │       ├── generators/
│       │       │   ├── DocumentationGeneratorTest.java
│       │       │   ├── AgentsMdGeneratorTest.java
│       │       │   ├── InstructionGeneratorTest.java
│       │       │   ├── PromptFileGeneratorTest.java
│       │       │   ├── CustomAgentGeneratorTest.java
│       │       │   ├── SkillGeneratorTest.java
│       │       │   └── VSCodeConfigGeneratorTest.java
│       │       ├── orchestrator/
│       │       │   └── RepoProcessorTest.java
│       │       └── report/
│       │           └── ReportGeneratorTest.java
│       └── resources/
│           └── fixtures/
│               ├── sample-pom.xml
│               ├── sample-build.gradle
│               ├── sample-config.yaml
│               └── sample-api-responses/
│
└── reports/                                                 # Generated reports (gitignored)
```

---

## 3. Phased Implementation Plan

### Phase 1: Foundation (Week 1-3)

**Goal:** Establish the Maven project, configuration system, Bitbucket API client, and Git operations.

#### Week 1: Project Setup and Configuration

| Task | Detail | Output |
|------|--------|--------|
| 1.1 Initialize Maven project | Create `pom.xml` with all dependencies, compiler plugin (Java 17+), shade plugin for executable JAR | Buildable project |
| 1.2 Config POJOs | Create all config classes (`AppConfig`, `BitbucketConfig`, `TargetConfig`, etc.) as Java records or standard POJOs | Config model |
| 1.3 ConfigLoader | YAML loading via Jackson YAML -> typed `AppConfig`; environment variable resolution via regex replacement in raw YAML | `ConfigLoader` class |
| 1.4 Data models | Create `AnalysisResult`, `AiContent`, `RepoResult`, `GeneratedFile`, `RunReport` as records | Model classes |
| 1.5 CLI entry point | `CopilotEnablerApp.java` with Picocli: `run` subcommand (config path, dry-run flag, verbosity) | `java -jar copilot-enabler.jar run --config config.yaml` |
| 1.6 Unit tests | Tests for ConfigLoader with valid/invalid YAML, missing env vars | `ConfigLoaderTest.java` |

#### Week 2: Bitbucket API Client

| Task | Detail | Output |
|------|--------|--------|
| 2.1 API response DTOs | `RepoDto`, `BranchDto`, `PullRequestDto`, `PagedResponse<T>` | DTO classes |
| 2.2 BitbucketApiClient | OkHttp-based client: `listRepos()`, `getRepo()`, `getDefaultBranch()`, `createBranch()`, `createPullRequest()` | `BitbucketApiClient` class |
| 2.3 BitbucketPaginator | Generic pagination: iterate `start`/`limit`/`isLastPage`/`nextPageStart` collecting all values | `BitbucketPaginator` class |
| 2.4 Authentication | HTTP Basic auth with user + token from environment variables; header injection via OkHttp interceptor | Auth interceptor |
| 2.5 Error handling | Map HTTP status codes to typed exceptions; retry on 429/503 with backoff | Exception hierarchy |
| 2.6 Unit tests | WireMock-based tests for all API operations including pagination and error cases | `BitbucketApiClientTest.java` |

#### Week 3: Git Operations

| Task | Detail | Output |
|------|--------|--------|
| 3.1 GitOperations class | JGit-based: `clone()`, `createBranch()`, `addAll()`, `commit()`, `push()`, `cleanup()` | `GitOperations` class |
| 3.2 Clone options | SSH and HTTPS support; configurable shallow clone depth; credentials provider for HTTPS | Clone implementation |
| 3.3 Branch handling | Create from base branch; handle existing branch (checkout + reset); detect default branch | Branch management |
| 3.4 SSH key handling | JGit `SshSessionFactory` configuration for SSH-based clone | SSH support |
| 3.5 Unit tests | Tests using `@TempDir` with `Git.init()` to create test repos | `GitOperationsTest.java` |

**Phase 1 deliverable:** The tool can read config, connect to Bitbucket, list repos, clone them, create branches, and push.

---

### Phase 2: Codebase Analysis Engine (Week 3-5)

**Goal:** Build the static analyzer (with native Maven POM parsing) and Copilot CLI integration.

#### Week 4: Static Analyzer

| Task | Detail | Output |
|------|--------|--------|
| 4.1 StaticAnalyzer interface | Defines `analyze(Path repoPath) -> AnalysisResult` | Interface |
| 4.2 MavenAnalyzer | Parse `pom.xml` using `org.apache.maven.model.io.xpp3.MavenXpp3Reader`; extract dependencies, plugins, properties, modules, Java version, profiles | `MavenAnalyzer` class |
| 4.3 GradleAnalyzer | Regex/pattern-based parsing of `build.gradle`/`build.gradle.kts` for dependencies, plugins, Java version | `GradleAnalyzer` class |
| 4.4 Framework detection | Detect Spring Boot, Quarkus, Micronaut from dependency group/artifact IDs | Framework identification |
| 4.5 Command extraction | Derive build/test/lint/run commands from build tool + detected plugins (e.g., `spotless-maven-plugin` -> `mvn spotless:apply`) | Command list |
| 4.6 Structure mapping | Walk directory tree via `java.nio.file.Files.walk()`; identify src, test, resource, migration dirs | Directory tree model |
| 4.7 Pattern detection | Detect controller-service-repository pattern, DTO classes, mapper interfaces, test frameworks, migration tool | Pattern list |
| 4.8 Unit tests | Tests against fixture `pom.xml` and `build.gradle` files; verify all extraction | `StaticAnalyzerTest.java`, `MavenAnalyzerTest.java` |

**Java advantage:** The `maven-model` library provides native, complete POM parsing including dependency management, profile resolution, and property interpolation -- significantly richer than XML parsing in Python.

#### Week 5: Copilot CLI Bridge and Prompt Engine

| Task | Detail | Output |
|------|--------|--------|
| 5.1 Freemarker prompt templates | Create 5 `.ftl` templates for Copilot CLI prompts (architecture, conventions, boundaries, workflows, commands) | Template files |
| 5.2 PromptTemplateEngine | Configure Freemarker; render templates with `AnalysisResult` data map | `PromptTemplateEngine` class |
| 5.3 CopilotCliBridge | `ProcessBuilder`-based invocation of `copilot -p`; capture stdout and stderr; configurable timeout | `CopilotCliBridge` class |
| 5.4 Response parsing | Parse markdown sections from Copilot CLI output; extract headings and content blocks | Parser methods |
| 5.5 Retry logic | Retry on `ProcessBuilder` timeout or non-zero exit code; exponential backoff | Retry wrapper |
| 5.6 Graceful degradation | If a prompt fails, return `Optional.empty()` for that dimension; log warning | Partial result handling |
| 5.7 Unit tests | Mock `ProcessBuilder` output; test parsing and error handling | `CopilotCliBridgeTest.java` |

**Phase 2 deliverable:** Given a cloned repo, the tool can produce a full `AnalysisResult` and AI content.

---

### Phase 3: Documentation and Agent File Generation (Week 5-7)

**Goal:** Build the two-phase generation pipeline: first generate technical documentation (`documents/`), then generate agent files that reference those documents.

#### Week 6: Documentation Generator (Phase A) and Core Agent Generators (Phase B)

| Task | Detail | Output |
|------|--------|--------|
| 6.1 FileGenerator interface | `List<GeneratedFile> generate(AnalysisResult, AiContent, Path repoPath, List<GeneratedFile> generatedDocs)` and `boolean shouldGenerate(AnalysisResult)` | Interface |
| 6.2 DocumentationGenerator | Generate `documents/` directory with detailed technical docs: `architecture.md`, `coding-conventions.md`, `api-design.md`, `testing-strategy.md`, `data-layer.md`, `build-and-commands.md`, `common-workflows.md`. Conditional: only create docs for detected patterns. Each doc contains specific code examples and real file paths | `DocumentationGenerator` class |
| 6.3 Documentation Freemarker templates | Create `.ftl` templates for each document type in `resources/templates/output/docs/` | Template files |
| 6.4 AgentsMdGenerator | Generate `AGENTS.md` with **mandatory context loading section** that lists all generated `documents/` files by path, instructing the agent to read them before any task. Combines static commands + AI persona/boundaries | Generator class |
| 6.5 InstructionGenerator | Generate `copilot-instructions.md` + conditional path-specific files based on detected patterns | Generator class |
| 6.6 VSCodeConfigGenerator | Generate/merge `.vscode/settings.json` (Jackson deep merge) and `extensions.json` | Generator class |
| 6.7 GeneratorRegistry | Discovers all `FileGenerator` implementations; runs DocumentationGenerator first (Phase A), then all others (Phase B) with the generated doc list | Registry class |
| 6.8 Unit tests | Test DocumentationGenerator (conditional doc creation, content structure); test AgentsMdGenerator (verify mandatory loading section lists generated docs) | Test classes |

#### Week 7: Advanced Generators and Idempotency

| Task | Detail | Output |
|------|--------|--------|
| 7.1 PromptFileGenerator | Generate `.prompt.md` files with YAML frontmatter, `${input:*}` variables, step-by-step instructions. Each prompt references relevant generated docs (e.g., `add-rest-endpoint.prompt.md` references `documents/api-design.md` and `documents/common-workflows.md`) | Generator class |
| 7.2 CustomAgentGenerator | Generate `.agent.md` files for test-specialist, api-reviewer, migration-helper (conditional). Each agent's instructions include directives to read relevant docs (e.g., `test-specialist` reads `documents/testing-strategy.md`) | Generator class |
| 7.3 SkillGenerator | Generate `SKILL.md` files in subdirectories for multi-step workflows (conditional). Skills reference relevant docs for context | Generator class |
| 7.4 JSON merge logic | Jackson-based deep merge for `settings.json`: read existing -> overlay new keys -> write; preserve non-Copilot settings | Utility method |
| 7.5 Markdown idempotency | Overwrite markdown files on re-run (generate fresh from latest analysis); optionally preserve user-added sections | Overwrite strategy |
| 7.6 Unit tests | Test conditional generation; test JSON merge with existing content; test idempotent re-run; verify agent files reference generated docs | Test classes |

**Phase 3 deliverable:** DocumentationGenerator produces detailed docs; all agent generators reference those docs; conditional generation and idempotency verified.

---

### Phase 4: Orchestration and PR Creation (Week 7-8)

**Goal:** Wire everything together with concurrency, PR creation, and reporting.

#### Week 8: Orchestrator, PR, and Report

| Task | Detail | Output |
|------|--------|--------|
| 8.1 RepoProcessor | Main orchestration loop: config -> discover repos -> process each (clone -> branch -> analyze -> generate -> commit -> push -> PR) | `RepoProcessor` class |
| 8.2 Concurrent execution | `ExecutorService` with configurable thread pool (or Virtual Threads on Java 21 via `Executors.newVirtualThreadPerTaskExecutor()`) | Concurrency |
| 8.3 PR description builder | Generate markdown description from `List<GeneratedFile>` organized by category (AGENTS.md, instructions, prompts, agents, skills, vscode) | Description builder |
| 8.4 PR creation | Invoke `BitbucketApiClient.createPullRequest()` with generated title and description | PR integration |
| 8.5 ReportGenerator | Aggregate `RepoResult` objects; write JSON via Jackson and Markdown summary to output directory | `ReportGenerator` class |
| 8.6 Dry-run mode | `--dry-run` flag: skip commit/push/PR; log what would be done | CLI flag |
| 8.7 Error handling | Per-repo try/catch; cleanup on failure; aggregate errors in report | Resilient processing |
| 8.8 End-to-end test | Full pipeline test with WireMock Bitbucket + temp git repos | Integration test |

**Phase 4 deliverable:** End-to-end execution: config -> discover -> process -> PRs -> report.

---

### Phase 5: Pilot and Measurement (Week 8-9)

**Goal:** Run against pilot repositories, collect metrics, iterate on quality.

| Task | Detail | Timeline |
|------|--------|----------|
| 9.1 Build executable JAR | `mvn clean package` producing a single executable JAR via shade plugin | Week 8 |
| 9.2 Baseline collection | Record agent task metrics and developer satisfaction survey | Week 8 |
| 9.3 Pilot run | Execute against <20 pilot repos; review generated PRs with teams | Week 8-9 |
| 9.4 Prompt iteration | Refine Freemarker prompt templates based on PR review feedback | Week 9 |
| 9.5 Template iteration | Refine output templates (AGENTS.md, prompt files, agent definitions) | Week 9 |
| 9.6 Re-run | Execute again after refinements; verify improved output | Week 9 |
| 9.7 Post-measurement | Collect metrics after 2-4 weeks of context file usage | Week 11-13 |
| 9.8 Scale-up plan | Document findings; prepare recommendation for scaling | Week 13 |

---

## 4. Key Implementation Details

### Configuration Loading

```java
public class ConfigLoader {
    private static final Pattern ENV_PATTERN = Pattern.compile("\\$\\{env:([^}]+)}");

    public AppConfig load(Path configPath) {
        String raw = Files.readString(configPath);
        String resolved = resolveEnvVars(raw);
        ObjectMapper yamlMapper = new ObjectMapper(new YAMLFactory());
        return yamlMapper.readValue(resolved, AppConfig.class);
    }

    private String resolveEnvVars(String raw) {
        Matcher matcher = ENV_PATTERN.matcher(raw);
        StringBuilder sb = new StringBuilder();
        while (matcher.find()) {
            String envVal = System.getenv(matcher.group(1));
            matcher.appendReplacement(sb, envVal != null ? envVal : "");
        }
        matcher.appendTail(sb);
        return sb.toString();
    }
}
```

### Maven POM Parsing (Java Advantage)

```java
public class MavenAnalyzer {
    public AnalysisResult analyze(Path repoPath) {
        Path pomPath = repoPath.resolve("pom.xml");
        MavenXpp3Reader reader = new MavenXpp3Reader();
        Model model = reader.read(Files.newInputStream(pomPath));

        List<String> dependencies = model.getDependencies().stream()
            .map(d -> d.getGroupId() + ":" + d.getArtifactId())
            .toList();

        String javaVersion = model.getProperties()
            .getProperty("java.version", "17");

        List<String> modules = model.getModules();

        boolean isSpringBoot = dependencies.stream()
            .anyMatch(d -> d.contains("spring-boot-starter"));

        // ... build AnalysisResult
    }
}
```

This is significantly richer than Python's XML parsing -- `maven-model` handles parent POM inheritance, property interpolation, profile activation, and dependency management sections natively.

### Copilot CLI Invocation

```java
public class CopilotCliBridge {
    public String invoke(String prompt, Path workingDir, int timeoutSeconds) 
            throws CopilotCliException {
        ProcessBuilder pb = new ProcessBuilder("copilot", "-p", prompt);
        pb.directory(workingDir.toFile());
        pb.redirectErrorStream(false);

        Process process = pb.start();
        String stdout = new String(process.getInputStream().readAllBytes());
        boolean finished = process.waitFor(timeoutSeconds, TimeUnit.SECONDS);

        if (!finished) {
            process.destroyForcibly();
            throw new CopilotCliException("Copilot CLI timed out after " 
                + timeoutSeconds + "s");
        }
        if (process.exitValue() != 0) {
            String stderr = new String(process.getErrorStream().readAllBytes());
            throw new CopilotCliException("Copilot CLI failed: " + stderr);
        }
        return stdout;
    }
}
```

### Generator Interface

```java
public interface FileGenerator {
    List<GeneratedFile> generate(
        AnalysisResult analysis,
        AiContent aiContent,
        Path repoPath
    );

    boolean shouldGenerate(AnalysisResult analysis);

    String category();
}
```

### Concurrent Processing with Virtual Threads (Java 21)

```java
public class RepoProcessor {
    public List<RepoResult> processAll(AppConfig config) {
        List<RepoInfo> repos = discoverRepos(config);
        List<RepoResult> results = Collections.synchronizedList(new ArrayList<>());

        try (var executor = Executors.newVirtualThreadPerTaskExecutor()) {
            List<Future<RepoResult>> futures = repos.stream()
                .map(repo -> executor.submit(() -> processRepo(repo, config)))
                .toList();

            for (Future<RepoResult> future : futures) {
                try {
                    results.add(future.get());
                } catch (ExecutionException e) {
                    results.add(RepoResult.failed(e.getCause()));
                }
            }
        }
        return results;
    }
}
```

For Java 17 (without Virtual Threads), replace with:

```java
ExecutorService executor = Executors.newFixedThreadPool(
    config.getProcessing().getConcurrency()
);
```

### JSON Deep Merge for settings.json

```java
public class JsonMerger {
    private final ObjectMapper mapper = new ObjectMapper()
        .enable(SerializationFeature.INDENT_OUTPUT);

    public String merge(String existing, String overlay) {
        ObjectNode existingNode = (ObjectNode) mapper.readTree(existing);
        ObjectNode overlayNode = (ObjectNode) mapper.readTree(overlay);
        deepMerge(existingNode, overlayNode);
        return mapper.writeValueAsString(existingNode);
    }

    private void deepMerge(ObjectNode target, ObjectNode source) {
        source.fields().forEachRemaining(entry -> {
            JsonNode targetVal = target.get(entry.getKey());
            if (targetVal != null && targetVal.isObject() 
                    && entry.getValue().isObject()) {
                deepMerge((ObjectNode) targetVal, (ObjectNode) entry.getValue());
            } else {
                target.set(entry.getKey(), entry.getValue());
            }
        });
    }
}
```

---

## 5. CLI Usage

```bash
# Build
mvn clean package

# Run with config
java -jar target/copilot-enabler.jar run --config config.yaml

# Dry run
java -jar target/copilot-enabler.jar run --config config.yaml --dry-run

# Verbose output
java -jar target/copilot-enabler.jar run --config config.yaml --verbose

# Single repo override
java -jar target/copilot-enabler.jar run --config config.yaml \
    --project PROJ1 --repo order-service
```

---

## 6. Maven POM Configuration (Key Sections)

```xml
<properties>
    <java.version>21</java.version>
    <maven.compiler.source>${java.version}</maven.compiler.source>
    <maven.compiler.target>${java.version}</maven.compiler.target>
    <okhttp.version>4.12.0</okhttp.version>
    <jgit.version>6.8.0.202311291450-r</jgit.version>
    <jackson.version>2.16.1</jackson.version>
    <freemarker.version>2.3.32</freemarker.version>
    <picocli.version>4.7.5</picocli.version>
</properties>

<dependencies>
    <!-- HTTP -->
    <dependency>
        <groupId>com.squareup.okhttp3</groupId>
        <artifactId>okhttp</artifactId>
        <version>${okhttp.version}</version>
    </dependency>

    <!-- Git -->
    <dependency>
        <groupId>org.eclipse.jgit</groupId>
        <artifactId>org.eclipse.jgit</artifactId>
        <version>${jgit.version}</version>
    </dependency>
    <dependency>
        <groupId>org.eclipse.jgit</groupId>
        <artifactId>org.eclipse.jgit.ssh.apache</artifactId>
        <version>${jgit.version}</version>
    </dependency>

    <!-- Config -->
    <dependency>
        <groupId>com.fasterxml.jackson.dataformat</groupId>
        <artifactId>jackson-dataformat-yaml</artifactId>
        <version>${jackson.version}</version>
    </dependency>
    <dependency>
        <groupId>com.fasterxml.jackson.core</groupId>
        <artifactId>jackson-databind</artifactId>
        <version>${jackson.version}</version>
    </dependency>

    <!-- Templates -->
    <dependency>
        <groupId>org.freemarker</groupId>
        <artifactId>freemarker</artifactId>
        <version>${freemarker.version}</version>
    </dependency>

    <!-- POM Parsing -->
    <dependency>
        <groupId>org.apache.maven</groupId>
        <artifactId>maven-model</artifactId>
        <version>3.9.6</version>
    </dependency>

    <!-- CLI -->
    <dependency>
        <groupId>info.picocli</groupId>
        <artifactId>picocli</artifactId>
        <version>${picocli.version}</version>
    </dependency>

    <!-- Logging -->
    <dependency>
        <groupId>org.slf4j</groupId>
        <artifactId>slf4j-api</artifactId>
        <version>2.0.11</version>
    </dependency>
    <dependency>
        <groupId>ch.qos.logback</groupId>
        <artifactId>logback-classic</artifactId>
        <version>1.4.14</version>
    </dependency>

    <!-- Test -->
    <dependency>
        <groupId>org.junit.jupiter</groupId>
        <artifactId>junit-jupiter</artifactId>
        <version>5.10.1</version>
        <scope>test</scope>
    </dependency>
    <dependency>
        <groupId>org.mockito</groupId>
        <artifactId>mockito-core</artifactId>
        <version>5.8.0</version>
        <scope>test</scope>
    </dependency>
    <dependency>
        <groupId>org.wiremock</groupId>
        <artifactId>wiremock-standalone</artifactId>
        <version>3.3.1</version>
        <scope>test</scope>
    </dependency>
    <dependency>
        <groupId>org.assertj</groupId>
        <artifactId>assertj-core</artifactId>
        <version>3.25.1</version>
        <scope>test</scope>
    </dependency>
</dependencies>
```

---

## 7. Key Differences from Python Approach

| Dimension | Python | Java |
|-----------|--------|------|
| **Timeline** | ~6-7 weeks | ~7-9 weeks (+2 weeks) |
| **Boilerplate** | Minimal (dynamic typing, concise syntax) | More (typed POJOs, checked exceptions, verbose generics) |
| **Type safety** | Optional (type hints + mypy) | Strong (compile-time verification) |
| **POM parsing** | Basic XML element tree | Native Maven model with inheritance and interpolation |
| **Concurrency** | `ThreadPoolExecutor` | Virtual Threads (Java 21) -- lighter, higher throughput |
| **Team fit** | May require Python upskilling | Matches existing Java team |
| **IDE support** | Good (PyCharm/VS Code) | Excellent (IntelliJ refactoring, error detection) |
| **Deployment** | Requires Python 3.10+ runtime | Single executable JAR (shade plugin) |
| **Testing** | pytest (concise) | JUnit 5 + Mockito (more verbose but familiar) |
| **Long-term maintenance** | Risk if team is Java-only | Natural fit |

### When to Choose Java

- The team will own and maintain this tool for years
- Type safety matters for reliability at scale (500+ repos)
- Richer POM parsing justifies the extra development time
- No throwaway work -- go straight to production quality
- Virtual Threads (Java 21) give excellent concurrency with minimal complexity

### When to Choose Python

- Speed of initial delivery is the priority
- Prompt engineering iteration (the highest-risk activity) benefits from fast turnaround
- The team is comfortable with Python or willing to learn
- The tool may remain a lightweight automation script, not a production platform

---

## 8. Risk Mitigation

| Risk | Mitigation |
|------|------------|
| Java boilerplate slows development | Use Java records for DTOs; Lombok if team accepts it; focus on value delivery over architecture |
| JGit SSH configuration complexity | Document SSH setup clearly; default to HTTPS clone; provide troubleshooting guide |
| Freemarker template debugging | Use strict mode; test templates independently with unit tests |
| Copilot CLI output quality varies | Multiple focused prompts; validate output; human review via PR |
| Team unfamiliar with Picocli | It has minimal API surface; Spring-boot-like annotation model |

---

## 9. Definition of Done

| Phase | Criteria |
|-------|----------|
| Phase 1 | Config loads from YAML; Bitbucket client lists repos (WireMock tested); JGit clone/branch/push works; >80% coverage |
| Phase 2 | `MavenAnalyzer` parses real pom.xml correctly; `GradleAnalyzer` handles build.gradle; `CopilotCliBridge` invokes and parses; >80% coverage |
| Phase 3 | All 6 generators produce valid files; conditional generation verified; JSON merge tested; >80% coverage |
| Phase 4 | End-to-end run: config -> discover -> process (concurrent) -> PRs -> report; dry-run works |
| Phase 5 | Pilot on <20 repos; baseline metrics collected; at least one prompt iteration; findings documented |
