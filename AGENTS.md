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
│   └── workflows/ci.yml           # Lint (JSON + markdown) for this repo
├── .editorconfig                   # Local formatting config
├── .prettierrc                     # Local formatting config
├── renovate.json                   # Extends sbaerlocher/.github:renovate-base
├── AGENTS.md / CLAUDE.md           # Agent instructions for this repo
└── README.md                       # GitHub profile README
```

## Related

- **Standards**: `sbaerlocher/.github` → `AGENTS.md`
- **Reusable Workflows**: `sbaerlocher/.github` → `.github/workflows/`
- **Renovate Presets**: `sbaerlocher/.github` → `renovate-*.json`
