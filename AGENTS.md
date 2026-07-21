# sbaerlocher - Profile Repository

## Context

GitHub profile repository (`sbaerlocher/sbaerlocher`). Hosts the profile README shown on
<https://github.com/sbaerlocher>. Shared standards, reusable workflows and Renovate presets
live in the central `sbaerlocher/.github` repository.

**Repository Type**: Profile
**Visibility**: Public

## Structure

```text
sbaerlocher/
├── .github/
│   ├── CODEOWNERS
│   ├── renovate.json               # Extends sbaerlocher/.github:renovate-base
│   └── workflows/
│       ├── pull-request.yml        # Lint (JSON + markdown) for this repo
│       └── weekly-security.yml     # Weekly security scan (config + secrets)
├── .editorconfig                   # Local formatting config
├── .prettierrc                     # Local formatting config
├── CONTRIBUTING.md                 # Contribution workflow
├── LICENSE                         # MIT
├── REVIEW.md                       # Code review guidelines
├── lefthook.yml                    # Local git hooks
├── AGENTS.md / CLAUDE.md           # Agent instructions for this repo
└── README.md                       # GitHub profile README
```

## Related

- **Standards**: `sbaerlocher/.github` → `AGENTS.md`
- **Reusable Workflows**: `sbaerlocher/.github` → `.github/workflows/`
- **Renovate Presets**: `sbaerlocher/.github` → `renovate-*.json`
