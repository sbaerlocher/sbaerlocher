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
- **Secrets** (see Secret Management section)

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

**Workflow Names (`name:` field):**

- Always write out full names (no abbreviations)
- Example: `name: Continuous Integration` (not `CI`)
- Example: `name: Security Scanning` (not `Security`)
- Match the workflow names in the table below

**Action References (Security):**

Actions MUST be pinned to full commit SHA with version comment:

```yaml
# ✅ CORRECT - SHA pinned with version comment
uses: actions/checkout@b4ffde65f46336ab88eb53be808477a3936bae11 # v4.1.1
uses: actions/setup-node@60edb5dd545a775178f52524783378180af0d1f8 # v4.0.2

# ❌ WRONG - Tag only (mutable, security risk)
uses: actions/checkout@v4
uses: actions/setup-node@v4.0.2
```

**Why SHA pinning?**

- **Immutable**: SHA cannot be changed after commit
- **Supply-chain security**: Protects against tag hijacking attacks
- **Audit trail**: Exact version is always traceable
- **GitHub recommendation**: For security-critical workflows

**Renovate handles updates**: With proper Renovate config, SHA references are automatically updated with new version comments.

**Renovate Configuration** (add to `renovate-base.json`):

```json
{
  "github-actions": {
    "pinDigests": true
  }
}
```

**Standard Workflows:**

| Workflow | Filename | Description |
|----------|----------|-------------|
| Continuous Integration | `ci.yml` | Tests, Linting, Validation, Type-Check, Coverage |
| Deployment | `deploy.yml` | Build & Deployment (incl. Terraform) |
| Release | `release.yml` | Semantic Release (Packages) |
| Security (Public) | `codeql.yml` | CodeQL Scanning (Multi-language, Public repos) |
| Security (IaC/GitOps) | `security.yml` | TFSec, Trivy, Kubesec, gitleaks |
| Security (Go) | `security.yml` | gosec, govulncheck |
| Security (Node.js) | `security.yml` | npm audit, Snyk |
| Security (Python) | `security.yml` | bandit, safety |
| Drift Detection | `drift.yml` | Scheduled Drift Detection (Terraform, optional) |
| Code Review (Private) | `claude-code-review.yml` | AI-assisted code review |
| AI Assistant (Private) | `claude.yml` | On-demand @claude mentions |

**Schedule Frequency:**

Scheduled workflows should run **weekly** (not daily) to minimize unnecessary workflow runs:

```yaml
schedule:
  - cron: '0 6 * * 1'  # Weekly Monday 06:00 UTC
```

Common schedules:

- `drift.yml`: Monday 06:00 UTC
- `security.yml`: Monday 02:00 UTC

**ci.yml Structure:**

See [templates/workflows/ci.yml](./templates/workflows/ci.yml)

**Benefits:**

- `workflow_call` enables reuse by other workflows
- `quality` and `test` run in parallel
- `build` only after both are green
- `format:check` validates formatting before other checks
- Faster feedback on failures

**By Repo Type:**

| Repo Type | Required | Optional | Notes |
|-----------|----------|----------|-------|
| **Infrastructure (IaC/GitOps)** | `ci.yml`, `deploy.yml`, `security.yml` | `drift.yml` | All validation in ci.yml |
| **Applications** | `ci.yml`, `deploy.yml` | - | Standard deployments |
| **Packages** | `ci.yml`, `release.yml` | - | NPM, Go, Python packages |

**Notes**:
- **ci.yml**: All validation (YAML, Fleet, Terraform, Helm, Kubernetes manifests, tests)
  - GitOps: yamllint, Fleet validation, Helm lint, kubeconform
  - IaC: yamllint, terraform fmt/validate, tflint
- **drift.yml**: Scheduled drift detection for Terraform (optional)
- **security.yml**: Language/stack-specific security scanning (required)
  - **IaC/GitOps**: TFSec, Trivy, Kubesec, gitleaks
  - **Go**: gosec, govulncheck
  - **Node.js**: npm audit, Snyk, ESLint security plugins
  - **Python**: bandit, safety, pip-audit
- **codeql.yml**: Multi-language SAST (required for public repos)

**Security Scanning Output:**

- **Public Repos**: SARIF upload to GitHub Security tab (free)
- **Private Repos**:
  - Artifact upload (JSON/Table format)
  - SARIF upload requires GitHub Advanced Security license
  - Use `exit-code: 0` for non-blocking scans

**Additionally for all repos**:

- **Public Repos**: `codeql.yml` (Required)
- **Private Repos**: `claude.yml` + `claude-code-review.yml` (Recommended)

### Claude GitHub Actions (Private Repos)

Two workflows for AI-assisted development:

**1. claude-code-review.yml - Automatic Code Review**

See [templates/workflows/claude-code-review.yml](./templates/workflows/claude-code-review.yml)

**2. claude.yml - On-Demand @claude Mentions**

See [templates/workflows/claude.yml](./templates/workflows/claude.yml)

**Setup:**

1. OAuth Token: <https://console.anthropic.com>
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

Shared presets in root can be extended by other repos.

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
  "extends": ["github>sbaerlocher/sbaerlocher:renovate-base"]
}
```

Or for JS/TS projects:

```json
{
  "extends": ["github>sbaerlocher/sbaerlocher:renovate-js"]
}
```

See [templates/](./templates/) for copy-paste examples.

**Note:** `lockFileMaintenance` is disabled by default. Normal dependency updates and `vulnerabilityAlerts` already cover transitive dependencies.

### Extended Documentation (Larger Projects)

For more complex projects additionally:

| File | Purpose | When |
|------|---------|------|
| `README.md` | Project overview, setup basics | All repos - keep minimal, avoid excessive detail |
| `ARCHITECTURE.md` | System design, components, decisions | Infrastructure, monorepos |
| `OPERATIONS.md` | Runbooks, troubleshooting, maintenance | Production services |
| `DEPLOYMENT.md` | Deploy process, environments | When more complex than 1 command |
| `.github/SECRETS.md` | Secrets documentation | **Required** when repo uses any secrets |

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

**.github/SECRETS.md Content:**

- Secret naming schema
- Architecture diagram (how secrets flow)
- GitHub repository secrets
- External secret store references (Bitwarden, Vault, etc.)
- Setup instructions
- Rotation checklist
- Troubleshooting

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

---

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

### YAML Linting

**Configuration**: `.yamllint.yml`

```yaml
extends: default

rules:
  line-length:
    max: 120
    level: warning
  indentation:
    spaces: 2
  comments:
    min-spaces-from-content: 1
```

**Usage**:

```bash
# Lint all YAML files
yamllint .

# Lint specific file
yamllint file.yaml

# Fix automatically (where possible)
yamllint --strict .
```

### Markdown Linting

**Configuration**: `.markdownlint.yml`

```yaml
# Extend default ruleset
default: true

# Line length
MD013:
  line_length: 120
  code_blocks: false
  tables: false

# Inline HTML
MD033: false

# Multiple headings with same content
MD024:
  siblings_only: true
```

**Usage**:

```bash
# Lint all markdown files
markdownlint '**/*.md'

# Lint specific file
markdownlint README.md

# Fix automatically
markdownlint --fix '**/*.md'
```

**Common Rules**:

- MD001: Heading levels increment by one
- MD013: Line length (120 characters)
- MD024: Multiple headings with same content
- MD025: Single title/H1 per document
- MD040: Fenced code blocks should have language

---

## Git Standards

### Commit Convention

**Format:** Conventional Commits with Claude Code signature

```text
<type>(<scope>): <subject>

<body>

🤖 Generated with [Claude Code](https://claude.com/claude-code)

Co-Authored-By: Claude <noreply@anthropic.com>
```

**Types:**

| Type | Usage | Description |
|------|-------|-------------|
| `feat` | New features | A new feature for the user |
| `fix` | Bug fixes | A bug fix for the user |
| `docs` | Documentation | Documentation only changes |
| `style` | Formatting | Changes that don't affect code meaning (white-space, formatting) |
| `refactor` | Code restructuring | Code change that neither fixes a bug nor adds a feature |
| `perf` | Performance | Code change that improves performance |
| `test` | Tests | Adding missing tests or correcting existing tests |
| `chore` | Maintenance | Changes to build process, dependencies, tooling |
| `ci` | CI/CD | Changes to CI configuration files and scripts |

**Scopes:**

Project-specific, examples:

- `infrastructure/platform` - Platform provisioning
- `functions/github-stats` - GitHub Stats function
- `applications/authentik` - Authentik service

**Examples:**

```text
feat(functions): add GitHub SVG generator endpoint

Implemented new endpoint to generate SVG badges for GitHub stats.
Includes caching and error handling.

🤖 Generated with [Claude Code](https://claude.com/claude-code)

Co-Authored-By: Claude <noreply@anthropic.com>
```

```text
fix(infrastructure): correct CPU normalization for Fleet

Changed CPU limits from "1000m" to "1" format to prevent
Fleet Modified status.

🤖 Generated with [Claude Code](https://claude.com/claude-code)

Co-Authored-By: Claude <noreply@anthropic.com>
```

### Branch Strategy

**Main Branch:** `main` (preferred) or `master`

**Strategy by Project Type:**

| Project Type | Strategy | Description |
|--------------|----------|-------------|
| **GitOps** | Direct to main | No feature branches, all changes direct after validation |
| **Infrastructure** | Direct to main | Fast deployment, CI/CD validation |
| **Applications** | Feature branches | Use feature branches for complex changes |
| **Packages** | Feature branches | Always use PRs for version-controlled releases |

**Direct-to-Main Requirements:**

- CI/CD validation pipeline must pass
- Auto-rollback on failure (GitOps)

**Feature Branch Naming:**

```text
<type>/<short-description>

Examples:
- feat/user-authentication
- fix/memory-leak
- chore/update-dependencies
```

**Tags/Releases:**

- Semantic Versioning: `v1.2.3`
- Annotated tags: `git tag -a v1.2.3 -m "Release 1.2.3"`
- Packages: Automated via `release.yml` workflow

---

## Secret Management

**CRITICAL**: Secrets must NEVER be committed to Git.

### Prohibited Patterns

**NEVER commit**:

- `.env` files with secrets
- `credentials.json`
- API keys in code
- Terraform `.tfvars` with secrets
- Kubernetes secrets in YAML
- SSH keys, certificates
- Database passwords

### Approved Secret Storage

| Environment | Method | Usage |
|-------------|--------|-------|
| **Kubernetes** | External Secrets Operator | All K8s secrets via Bitwarden |
| **CI/CD** | GitHub Repository Secrets | Workflow secrets |
| **Cloudflare Workers** | Wrangler Secrets | `wrangler secret put <NAME>` |
| **Local Development** | `.env.local` (gitignored) | Never committed |

### External Secrets Operator Pattern

**For Kubernetes services**:

```yaml
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata:
  name: service-credentials
  namespace: service
spec:
  refreshInterval: 15m
  secretStoreRef:
    name: bitwarden-secretstore
    kind: ClusterSecretStore
  target:
    name: service-secret
    creationPolicy: Owner
  data:
    - secretKey: password
      remoteRef:
        key: <bitwarden-item-id>
        property: password
```

### Secret Naming Schema

**Standard across all repositories** (Bitwarden Secrets Manager):

```text
<Project> | <Service/Component> | <Verwendungsort/Scope> | <Permission> | <Type>
```

**Field Descriptions:**

- **Project**: Main project using this secret (e.g., `Functions`, `Infrastructure`, `Authentication`, `OpenArchiver`)
- **Service/Component**: External service or component (e.g., `GitHub API`, `Cloudflare DNS`, `Storj S3`)
- **Verwendungsort/Scope**: Where the secret is used (e.g., `nuvulus-cluster`, `sbaerlocher/functions`, `uptime-service`)
- **Permission**: Permission level (e.g., `read-only`, `read-write`, `admin`, `confidential`)
- **Type**: Credential type (e.g., `API Token`, `Access Key`, `Secret Key`, `Client Secret`)

**Important**: Not all fields are required for every secret. Field count is flexible, but order must be consistent.

**Examples:**

```text
# GitOps/Kubernetes
Infrastructure | Hetzner Cloud | nuvulus-cluster | read-write | API Token
Database | PostgreSQL | nuvulus-cluster | superuser | Password
OpenArchiver | Storj S3 | storage | read-write | Access Key

# Cloudflare Workers
Functions | Cloudflare Workers Deploy | sbaerlocher/functions | edit | API Token
Functions | GitHub API | sbaerlocher/functions repository | read-only | Personal Access Token
Functions | Grafana Synthetic Monitoring | uptime-service | read | API Token

# Terraform
Terraform | Cloudflare R2 | all-projects | read-write | Access Key ID
Authentication | Authentik API | terraform | admin | API Token
```

### Secret Scanning

**Automated Detection**:

- **CI/CD**: gitleaks action in workflows
- **GitHub**: Secret scanning alerts (public repos)

**gitleaks Configuration**:

See [templates/.gitleaks.toml](./templates/.gitleaks.toml)

### .gitignore for Secrets

**Required entries**:

```gitignore
# Secrets
.env
.env.local
.env.*.local
*.pem
*.key
*.crt
credentials.json
secrets.yaml
*-credentials.json

# Terraform
*.tfvars
!example.tfvars
terraform.tfstate
terraform.tfstate.backup

# Kubernetes
kubeconfig
*-secret.yaml
!*-secret.example.yaml
```

### Secret Rotation

**Regular rotation required for**:

- Database passwords (quarterly)
- API keys (semi-annually)
- Certificates (before expiry)
- Service account tokens (annually)

**Process**:

1. Generate new secret
2. Update in secret store (Bitwarden)
3. External Secrets refreshes automatically
4. Verify services still functional
5. Deactivate old secret

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
| `/actions-check` | `actions-check.md` | GitHub Actions best practices (SHA pinning, permissions, etc.) |

**Templates**: See [templates/commands/](./templates/commands/) for copy-paste examples.

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

## Kubernetes Resource Standards

**For GitOps and Kubernetes projects**

### Resource Limits & Requests

**CRITICAL**: CPU values >= 1000m must use normalized string format

### Size Classes

Use standardized resource sizes:

| Size | CPU Requests | CPU Limits | Memory Requests | Memory Limits |
|------|--------------|------------|-----------------|---------------|
| **Micro** | 50m | 250m | 64Mi | 256Mi |
| **Small** | 100m | 500m | 128Mi | 512Mi |
| **Medium** | 200m | "1" | 256Mi | 1Gi |
| **Large** | 500m | "2" | 512Mi | 2Gi |
| **XLarge** | "1" | "4" | 1Gi | 4Gi |

### CPU Normalization

**Why**: Kubernetes normalizes CPU values internally, causing Fleet "Modified" status

**Rules**:

```yaml
# ✅ CORRECT
resources:
  limits:
    cpu: "1"      # >= 1000m use string format
    cpu: "2"      # >= 1000m use string format
    memory: 2Gi
  requests:
    cpu: 500m     # < 1000m can use millicores
    memory: 512Mi

# ❌ WRONG
resources:
  limits:
    cpu: 1000m    # Will be normalized to "1" → Fleet drift!
    cpu: 2000m    # Will be normalized to "2" → Fleet drift!
```

### Deployment Strategies

**Resource-Constrained Clusters** (CPU >95%):

```yaml
spec:
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 0        # No extra pods during update
      maxUnavailable: 1  # Old pod terminated first
```

**Standard Clusters** (CPU <90%):

```yaml
spec:
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1        # One extra pod during update
      maxUnavailable: 0  # Zero downtime
```

### Service Assignments

**By Service Type**:

- **Large**: Databases (PostgreSQL), Ingress (Traefik)
- **Medium**: External Secrets, Monitoring, Authentik, n8n
- **Small**: Cert-Manager, Operators, External-DNS
- **Micro**: Reflector, lightweight operators

### Health Checks

**Required for all services**:

```yaml
livenessProbe:
  httpGet:
    path: /health
    port: 8080
  initialDelaySeconds: 30
  periodSeconds: 10
  failureThreshold: 3

readinessProbe:
  httpGet:
    path: /ready
    port: 8080
  initialDelaySeconds: 5
  periodSeconds: 5
  failureThreshold: 3
```

---

## Monitoring Standards

**For services with metrics**

### Grafana Alloy Annotations

**Required for Prometheus scraping**:

```yaml
metadata:
  annotations:
    k8s.grafana.com/scrape: "true"
    k8s.grafana.com/metrics.portNumber: "9187"
    k8s.grafana.com/metrics.path: "/metrics"
    k8s.grafana.com/job: "<service-name>"
    k8s.grafana.com/metrics.scrapeInterval: "60s"  # Optional
```

### Common Metrics Ports

| Service | Port | Path | Key Metrics |
|---------|------|------|-------------|
| **External Secrets** | 8080 | /metrics | `externalsecret_status_condition` |
| **Cert-Manager** | 9402 | /metrics | `certmanager_certificate_ready_status` |
| **PostgreSQL (CNPG)** | 9187 | /metrics | `cnpg_pg_database_size_bytes` |
| **Traefik** | 9100 | /metrics | `traefik_service_requests_total` |
| **Alloy** | 12345 | /metrics | `prometheus_target_scrapes_total` |

### Metrics Naming Convention

**Format**: `<namespace>_<resource>_<metric>_<unit>`

**Examples**:

- `http_requests_total` (counter)
- `http_request_duration_seconds` (histogram)
- `database_connections_active` (gauge)
- `cache_hits_total` (counter)

### Alert Thresholds

**Standard SLOs**:

- **Availability**: 99.9% (max 43.8 min downtime/month)
- **Latency (p95)**: < 500ms
- **Error Rate**: < 1%
- **Saturation**: < 80% CPU/Memory

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
