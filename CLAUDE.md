# CLAUDE.md

This file provides guidance to AI assistants (Claude and others) working in this repository. It documents project conventions, development workflows, and important context to ensure consistent and effective contributions.

---

## Repository Overview

**Repository**: jaybinary/Claude
**Branch model**: Feature branches prefixed with `claude/` for AI-assisted work
**Status**: Active development

> Update this section with a brief description of what this project does, its primary purpose, and the intended audience.

---

## Project Structure

```
.
├── CLAUDE.md           # This file — AI assistant guidance
├── README.md           # Human-facing project documentation
├── .github/
│   └── workflows/      # CI/CD pipeline definitions
├── src/                # Primary source code
├── tests/              # Test suites
├── docs/               # Extended documentation
└── scripts/            # Utility and automation scripts
```

> Update this tree as the actual project structure evolves.

---

## Development Setup

### Prerequisites

Document required tools and versions here. Example:

```bash
# Example — replace with actual requirements
node >= 18
python >= 3.11
go >= 1.21
```

### Installation

```bash
# Clone and set up the project
git clone <repo-url>
cd Claude

# Install dependencies (update with actual command)
# npm install
# pip install -r requirements.txt
# go mod download
```

### Environment Variables

List required environment variables. Never commit secrets. Example:

```bash
# Copy and fill in your values
cp .env.example .env
```

| Variable | Required | Description |
|----------|----------|-------------|
| `API_KEY` | Yes | API key for external service |
| `DATABASE_URL` | Yes | Connection string for database |
| `LOG_LEVEL` | No | Logging verbosity (default: `info`) |

---

## Common Commands

Update this section with the actual commands for this project.

### Build

```bash
# Build the project
# npm run build
# make build
# go build ./...
```

### Test

```bash
# Run all tests
# npm test
# pytest
# go test ./...

# Run tests with coverage
# npm run test:coverage
# pytest --cov=src
# go test -cover ./...

# Run a single test file or pattern
# npm test -- --testPathPattern=<pattern>
# pytest tests/test_specific.py
```

### Lint & Format

```bash
# Lint the codebase
# npm run lint
# ruff check .
# golangci-lint run

# Auto-fix lint errors
# npm run lint:fix
# ruff check --fix .

# Format code
# npm run format
# black .
# gofmt -w .
```

### Run Locally

```bash
# Start the development server
# npm run dev
# python -m uvicorn main:app --reload
# go run ./cmd/server
```

---

## Git Workflow

### Branch Naming

| Branch type | Pattern | Example |
|-------------|---------|---------|
| AI-assisted features | `claude/<description>-<session-id>` | `claude/add-auth-abc123` |
| Human features | `feature/<description>` | `feature/add-auth` |
| Bug fixes | `fix/<issue-or-description>` | `fix/null-pointer-on-login` |
| Releases | `release/<version>` | `release/v1.2.0` |

### Commit Conventions

Follow [Conventional Commits](https://www.conventionalcommits.org/):

```
<type>(<scope>): <short summary>

[optional body]

[optional footer]
```

**Types**: `feat`, `fix`, `docs`, `style`, `refactor`, `test`, `chore`, `perf`, `ci`

**Examples**:
```
feat(auth): add OAuth2 login flow
fix(api): handle null response from upstream service
docs: update CLAUDE.md with new commands
test(user): add coverage for edge cases in signup
```

**Rules**:
- Use imperative mood in subject line ("add feature" not "added feature")
- Keep subject line under 72 characters
- Reference issues in footer: `Closes #42`

### Pull Request Process

1. Branch off `main` (or the designated base branch)
2. Make focused, reviewable changes
3. Ensure all tests pass and linting is clean before opening a PR
4. Write a clear PR description explaining the *why* behind changes
5. Link related issues

---

## Code Conventions

> Replace the examples below with language/framework-specific conventions actually used in this project.

### General Principles

- **Simplicity first**: Prefer readable, direct code over clever abstractions
- **No premature optimization**: Optimize only when profiling confirms a bottleneck
- **Fail loudly**: Raise/return errors explicitly; avoid silent failures
- **Test behavior, not implementation**: Tests should document what the system does, not how

### Naming

- Variables and functions: `camelCase` (JS/TS) / `snake_case` (Python/Go)
- Constants: `SCREAMING_SNAKE_CASE`
- Classes/types: `PascalCase`
- Files: Match the primary export name (e.g., `UserService.ts` exports `UserService`)

### Error Handling

- Always handle errors at the call site; do not swallow exceptions
- Use typed errors / custom exception classes where applicable
- Log errors with sufficient context (operation, input identifiers, error message)

### Comments

- Write comments to explain *why*, not *what* (code should explain what)
- Mark temporary workarounds with `// TODO(<author>): <description>` and a linked issue
- Avoid commented-out code in commits — use `git stash` or a branch instead

---

## Testing

### Test Organization

```
tests/
├── unit/          # Fast, isolated unit tests
├── integration/   # Tests involving real I/O or multiple components
└── e2e/           # End-to-end tests against a running system
```

### What to Test

- All public API surface area
- Error paths and edge cases, not just the happy path
- Behavior that would be non-obvious to a future reader

### Test Hygiene

- Each test should be independent and idempotent (can run in any order, any number of times)
- Use test fixtures/factories for repeated setup
- Mock external services at the boundary — avoid network calls in unit tests
- Prefer descriptive test names: `should return 404 when user does not exist`

---

## CI/CD

> Document the actual pipeline stages once CI is configured.

The CI pipeline (`.github/workflows/`) runs on every pull request and push to `main`:

| Stage | Description |
|-------|-------------|
| **lint** | Static analysis and formatting checks |
| **test** | Unit and integration test suites |
| **build** | Compile/bundle the project |
| **deploy** | Deploy to staging (on merge to `main`) |

All stages must pass before a PR can be merged.

---

## AI Assistant Guidelines

### Before Making Changes

1. **Read before writing**: Always read the files you plan to modify before editing
2. **Understand context**: Check related files, imports, and usages before changing an interface
3. **Check tests**: Find and read existing tests for code you plan to change
4. **Look for patterns**: Follow existing conventions in the codebase rather than introducing new ones

### When Implementing

- Make the minimal change that satisfies the requirement — avoid scope creep
- Do not add unrequested features, refactors, or "improvements"
- Do not add comments or docstrings to code you did not touch
- Prefer editing existing files over creating new ones
- Never introduce security vulnerabilities (SQL injection, XSS, command injection, etc.)
- Do not commit secrets, credentials, or environment-specific values

### Verification Checklist

Before committing, verify:
- [ ] All existing tests still pass
- [ ] New behavior is covered by tests
- [ ] Linting passes with no new warnings
- [ ] No debug/temporary code left in files
- [ ] Commit message follows Conventional Commits format

### What to Avoid

- Backward-compatibility shims for code that no longer exists
- Over-abstraction for single-use patterns
- Error handling for scenarios that cannot occur
- Unnecessary logging or console output
- Ignoring or suppressing lint warnings without justification

---

## Security

- Never log sensitive data (passwords, tokens, PII)
- Validate all input at system boundaries (user input, external API responses)
- Use parameterized queries for all database access
- Keep dependencies up to date; run `npm audit` / `pip-audit` / `govulncheck` regularly
- Follow the principle of least privilege for service accounts and API keys

---

## Troubleshooting

> Populate this section with common issues and their solutions as they are encountered.

### Problem: Tests fail with connection errors

**Likely cause**: External service dependency not running or not mocked
**Solution**: Ensure Docker Compose services are up (`docker compose up -d`) or check that mocks are correctly configured

### Problem: Build fails after dependency update

**Likely cause**: Incompatible version or missing peer dependency
**Solution**: Run `npm install` / `pip install -r requirements.txt` again; check the dependency changelog for breaking changes

---

## Maintainers

> List the primary maintainers and their areas of ownership.

| Name | Area |
|------|------|
| @jaybinary | Project owner |

---

*This CLAUDE.md was generated on 2026-02-20. Keep it up to date as the project evolves — an accurate CLAUDE.md is one of the highest-leverage contributions you can make to a codebase.*
