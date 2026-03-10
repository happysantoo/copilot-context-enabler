# Contributing to Copilot Context Enabler

Thank you for your interest in contributing. This document provides guidelines and information for contributors.

## How to Contribute

### Reporting Issues

- Use the GitHub issue tracker to report bugs or request features
- Include steps to reproduce for bug reports
- Describe the expected vs actual behavior
- Include your environment details (OS, Java/Python version, Bitbucket Server version)

### Submitting Changes

1. Fork the repository
2. Create a feature branch from `main` (`git checkout -b feature/your-feature`)
3. Make your changes
4. Add or update tests as appropriate
5. Ensure all tests pass
6. Commit with a clear message describing the change
7. Push to your fork and open a pull request against `main`

### Pull Request Guidelines

- Keep PRs focused on a single concern
- Include a description of what the PR does and why
- Reference any related issues
- Ensure CI checks pass before requesting review
- Be responsive to review feedback

## Development Setup

### Prerequisites

- Git
- GitHub Copilot CLI (for integration testing)
- Access to a Bitbucket Server 7.x instance (for end-to-end testing)

### Python Implementation

```bash
# Create virtual environment
python -m venv .venv
source .venv/bin/activate

# Install in development mode
pip install -e ".[dev]"

# Run tests
pytest

# Run linting
ruff check .

# Run type checking
mypy src/
```

### Java Implementation

```bash
# Build
mvn clean verify

# Run tests only
mvn test

# Run with specific test
mvn test -Dtest=BitbucketApiClientTest
```

## Code Style

### Python

- Follow PEP 8
- Use type hints for all function signatures
- Format with `ruff format`
- Lint with `ruff check`
- Maximum line length: 100 characters

### Java

- Follow standard Java conventions
- Use meaningful variable and method names
- Write Javadoc for public APIs
- Format consistently (consider using `spotless` or `google-java-format`)

## Testing

- Write unit tests for all new functionality
- Use mocks for external dependencies (Bitbucket API, Copilot CLI, Git)
- Aim for >80% code coverage on new code
- Integration tests should be clearly marked and runnable separately

## Project Structure

Refer to the implementation plan documents for detailed project structure:
- [Python Implementation Plan](documents/implementation-plan-python.md)
- [Java Implementation Plan](documents/implementation-plan-java.md)

## Areas for Contribution

- **Prompt engineering**: Improve the Copilot CLI prompts for better analysis output
- **Output templates**: Refine the generated context files for better Agent Mode results
- **New generators**: Add generators for additional file types or patterns
- **Language support**: Extend static analysis beyond Java/Maven/Gradle
- **Bitbucket Cloud support**: Add support for Bitbucket Cloud API alongside Server
- **Other Git platforms**: Add support for GitHub, GitLab
- **Documentation**: Improve docs, add examples, write tutorials

## License

By contributing, you agree that your contributions will be licensed under the Apache License 2.0 that covers this project. See [LICENSE](LICENSE) for details.
