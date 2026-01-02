# Repository Standards

## Principles

**Public vs Private Repos:**

| Aspect | Public | Private |
|--------|--------|---------|
| LICENSE | MIT (Required) | None (Rights with Owner) |
| `codeql.yml` | Required | Optional |
| Claude Workflows | Not recommended | Recommended |
| Other structure | Identical | Identical |

---

## Recommended Minimal Structure

Focus on essentials - no unnecessary files.

```text
repository/
├── .github/
│   ├── workflows/
│   │   └── [project-specific].yml
│   ├── CODEOWNERS
│   └── renovate.json
├── AGENTS.md                      # AI instructions (main file)
├── CLAUDE.md                      # Only: @AGENTS.md
├── README.md                      # Main documentation
├── LICENSE                        # MIT License (public only)
├── .editorconfig                  # Editor consistency
├── .gitignore                     # Project-specific
└── [project files]
```

---

## Required Files

### 1. README.md

**Content (minimal):**

```markdown
# Project Name

Short description (1-2 sentences).

## Quick Start

[Commands to start]

## Structure

[Brief folder overview if needed]

## License

MIT License
```

**Avoid:**

- Badges (except npm/go packages)
- Long feature lists
- Code examples (belong in AGENTS.md or docs)

### 2. LICENSE (Public Repos Only)

MIT License for all public repos. Private repos don't need LICENSE (rights with owner).

```
MIT License

Copyright (c) [YEAR] [YOUR NAME]

Permission is hereby granted...
```

### 3. .editorconfig

See [templates/.editorconfig](./templates/.editorconfig)

### 4. .gitignore

Project-specific, keep minimal:

- Build outputs
- Dependencies (node_modules, .terraform)
- IDE files (.idea, .vscode)
- OS files (.DS_Store)
- Secrets (.env, *.tfvars with secrets)

### 5. .github/CODEOWNERS

```text
# Default owner
* @your-username
```

### 6. .github/renovate.json

Required for all repos. See base configuration in [templates/renovate-base.json](./templates/renovate-base.json).

---

## Optional (Only When Useful)

### CHANGELOG.md

Only for:

- npm/go packages with versions
- Projects with releases

Format: Keep a Changelog

```markdown
# Changelog

## [1.0.0] - 2025-01-15
### Added
- Feature X
### Fixed
- Bug Y
```

### .github/workflows/

**Naming Convention:**

- Lowercase
- Hyphens instead of underscores
- Short and descriptive

**Standard Workflows:**

| Workflow | Filename | Description |
|----------|----------|-------------|
| Continuous Integration | `ci.yml` | Tests, Linting, Type-Check, Coverage |
| Deployment | `deploy.yml` | Build & Deployment (incl. Terraform) |
| Release | `release.yml` | Semantic Release (Packages) |
| Security | `codeql.yml` | Security Scanning (Public only) |
| Validation | `validate.yml` | YAML/Config Validation (GitOps) |
| Drift Detection | `drift.yml` | Scheduled Drift Detection (IaC) |

**ci.yml Structure:**

See [templates/workflows/ci.yml](./templates/workflows/ci.yml)

**Benefits:**

- `workflow_call` enables reuse by other workflows
- `quality` and `test` run in parallel
- `build` only after both are green
- `format:check` validates formatting before other checks
- Faster feedback on failures

**By Repo Type:**

| Repo Type | Workflows |
|-----------|-----------|
| Terraform/IaC | `deploy.yml`, `drift.yml` |
| Applications | `deploy.yml`, `ci.yml` |
| Packages | `ci.yml`, `release.yml` |
| GitOps | `validate.yml` |

Additionally for **Public Repos**: `codeql.yml` (Required)

Additionally for **Private Repos**: `claude.yml` + `claude-code-review.yml` (Recommended)

### Claude GitHub Actions (Private Repos)

Two workflows for AI-assisted development:

**1. claude-code-review.yml - Automatic Code Review**

See [templates/workflows/claude-code-review.yml](./templates/workflows/claude-code-review.yml)

**2. claude.yml - On-Demand @claude Mentions**

See [templates/workflows/claude.yml](./templates/workflows/claude.yml)

**Setup:**

1. OAuth Token: https://console.anthropic.com
2. Secret: Repository → Settings → Secrets → `CLAUDE_CODE_OAUTH_TOKEN`

**Why Private Repos Only?**

- OAuth Token is personal/paid
- On public repos anyone could trigger `@claude`

### Testing & Coverage

**Test Framework:** Vitest (for JS/TS projects)

**package.json Scripts:**

```json
{
  "scripts": {
    "test": "vitest run",
    "test:watch": "vitest",
    "test:coverage": "vitest run --coverage"
  }
}
```

**vitest.config.ts:**

See [templates/vitest.config.ts](./templates/vitest.config.ts)

**Dependencies:**

```json
{
  "devDependencies": {
    "vitest": "^3.0.0",
    "@vitest/coverage-v8": "^3.0.0"
  }
}
```

**Codecov:** Coverage upload in CI with `codecov/codecov-action`.

### Renovate Configuration

Shared presets in [.github/](./.github/) can be extended by other repos.

**Available Presets:**

| Preset | Extends | Use For |
|--------|---------|---------|
| `renovate-base` | - | All repos (base config) |
| `renovate-js` | base | JS/TS projects |
| `renovate-terraform` | base | Terraform/IaC |
| `renovate-gitops` | base | GitOps/Helm |

**Usage in your repo's `.github/renovate.json`:**

```json
{
  "extends": ["local>sbaerlocher/sbaerlocher:renovate-base"]
}
```

Or for JS/TS projects:

```json
{
  "extends": ["local>sbaerlocher/sbaerlocher:renovate-js"]
}
```

See [templates/](./templates/) for copy-paste examples.

**Note:** `lockFileMaintenance` is disabled by default. Normal dependency updates and `vulnerabilityAlerts` already cover transitive dependencies.

### Extended Documentation (Larger Projects)

For more complex projects additionally:

| File | Purpose | When |
|------|---------|------|
| `ARCHITECTURE.md` | System design, components, decisions | Infrastructure, monorepos |
| `OPERATIONS.md` | Runbooks, troubleshooting, maintenance | Production services |
| `DEPLOYMENT.md` | Deploy process, environments | When more complex than 1 command |

**Naming:** Uppercase, English.

**ARCHITECTURE.md Content:** (from architect perspective)

- System overview (Mermaid diagram)
- Components and interfaces
- Tradeoffs and limitations
- Security concept

**OPERATIONS.md Content:**

- Monitoring & Alerts
- Troubleshooting guide
- Maintenance tasks
- Incident response

### CONTRIBUTING.md

Only for:

- Open source projects
- Team projects with external contributors

For personal projects: Not needed.

### CODE_OF_CONDUCT.md

Only for public community projects. For personal repos: Not needed.

---

## What Does NOT Belong in Every Repo

| File | Reason |
|------|--------|
| docs/ folder | README is usually enough, no separate doc site |
| SECURITY.md | Only for public packages |
| Issue/PR Templates | Overkill for personal repos |

## Formatter by Language

| Language | Formatter | Config File |
|----------|-----------|-------------|
| JS/TS | Prettier | `.prettierrc` |
| Go | gofmt | - (built-in) |
| Terraform | terraform fmt | - (built-in) |
| Python | Black/Ruff | `pyproject.toml` |
| YAML | Prettier | `.prettierrc` |

Only add when the language is used in the repo.

### Prettier Configuration (JS/TS)

**.prettierrc:**

See [templates/.prettierrc](./templates/.prettierrc)

**.prettierignore:**

See [templates/.prettierignore](./templates/.prettierignore)

**package.json Scripts:**

```json
{
  "scripts": {
    "format": "prettier --write .",
    "format:check": "prettier --check ."
  }
}
```

**Dependencies:**

```json
{
  "devDependencies": {
    "prettier": "^3.0.0",
    "prettier-plugin-astro": "^0.14.0",
    "prettier-plugin-svelte": "^3.0.0"
  }
}
```

Add plugins as needed per framework (astro, svelte, tailwind, etc.).

---

## AI Agent Documentation

AI instructions belong in a separate file, **not** in README.

**Recommended Structure:**

```text
repository/
├── AGENTS.md              # AI instructions (main file)
├── CLAUDE.md              # Only import: @AGENTS.md
├── .claude/
│   ├── commands/          # Custom Slash Commands (optional)
│   └── settings.local.json
└── README.md              # Human-readable docs
```

**CLAUDE.md Content (minimal):**

```markdown
@AGENTS.md
```

**AGENTS.md Content:**

- Project context for AI
- Code conventions
- Architecture overview
- Important patterns
- What AI should not do

**Why this approach?**

- AGENTS.md is the emerging standard (20,000+ repos)
- Claude Code currently only supports CLAUDE.md
- With `@AGENTS.md` import it works today
- Future-proof when AGENTS.md is officially supported

### .claude/commands/

Custom Slash Commands for recurring tasks.

**Standard Commands (all repos):**

| Command | File | Description |
|---------|------|-------------|
| `/code-review` | `code-review.md` | Code review for file/folder |
| `/commit` | `commit.md` | Git commit with Conventional Commits |
| `/quality-check` | `quality-check.md` | All checks before PR |
| `/actions-check` | `actions-check.md` | GitHub Actions best practices |

**Project-specific Commands:**

| Repo Type | Commands |
|-----------|----------|
| Terraform | `tf-plan.md`, `tf-apply.md` |
| GitOps | `fleet-check.md`, `helm-check.md` |
| Cloudflare Workers | `deploy.md`, `new-service.md` |
| Packages | `release.md`, `test.md` |

**Format:**

```markdown
---
description: Short description for /help
---

Instructions for Claude...

$ARGUMENTS = Parameters from user
```

---

## Template for New Repos

```text
new-repo/
├── .github/
│   ├── CODEOWNERS
│   └── renovate.json
├── AGENTS.md
├── CLAUDE.md
├── .editorconfig
├── .gitignore
├── LICENSE                    # public repos only
├── README.md
└── [project files]
```

**CLAUDE.md:**

```markdown
@AGENTS.md
```

**AGENTS.md:**

```markdown
# Project Name

## Context

[What the project does, for AI]

## Conventions

[Code style, patterns]

## Structure

[Important folders/files]
```

**README.md:**

```markdown
# project-name

What the project does (1 sentence).

## Setup

\`\`\`bash
# Installation/start commands
\`\`\`

## License

MIT
```

---

## Summary

**Principles:**

1. Less is more
2. README is the main documentation
3. LICENSE always included
4. CHANGELOG only for releases
5. Workflows only what's needed
6. No template files that stay empty
