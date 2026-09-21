# workflows-backend

Reusable GitHub Actions workflows for backend PR pipelines, independent of the language or toolchain in use. The caller repository supplies its own shell commands; this repository only orchestrates the sequence and enforces failures.

## Usage

```yaml
# .github/workflows/pull_request.yml
name: Workflow PR

on:
  pull_request:
    branches: [main]

jobs:
  pipeline:
    uses: juv-dev/workflows-backend/.github/workflows/backend-pr-pipeline.yml@main
    with:
      setup-command: "pnpm install --frozen-lockfile"
      dependency-audit-command: "pnpm audit --audit-level high --prod"
      lint-command: "pnpm run lint"
      build-command: "pnpm run build"
      test-command: "pnpm run test:coverage"
      coverage-check-command: "pnpm run coverage:check -- --min 80"
      version-command: "node -p \"require('./package.json').version\""
```

Any language works as long as the caller provides equivalent commands, for example Python:

```yaml
    with:
      setup-command: "pip install -r requirements.txt"
      dependency-audit-command: "pip-audit"
      lint-command: "ruff check ."
      build-command: "python -m compileall ."
      test-command: "pytest --cov --cov-report=term-missing"
      coverage-check-command: "coverage report --fail-under=80"
      version-command: "poetry version -s"
```

`coverage-check-command` and `version-command` are optional; omit them if the project has no coverage gate or does not publish tagged releases.

## Security scan only

For push/schedule-triggered scanning without the full PR pipeline:

```yaml
jobs:
  security:
    uses: juv-dev/workflows-backend/.github/workflows/backend-security-scan.yml@main
    with:
      setup-command: "pnpm install --frozen-lockfile"
      dependency-audit-command: "pnpm audit --audit-level high --prod"
```
