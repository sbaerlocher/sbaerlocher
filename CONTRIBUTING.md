# Contributing

## Workflow

All changes go through a feature branch and pull request. Do not commit
directly to `main`.

Use a focused branch name such as `fix/workflow-permissions` or
`chore/renovate-preset-cleanup`.

## Commits

Commit messages follow Conventional Commits:

```text
<type>(<scope>): <subject>
```

Common types are `feat`, `fix`, `docs`, `ci`, `chore`, `refactor`, `test`,
and `style`. Keep the subject lowercase, specific, and without trailing
punctuation.

Sign and sign off commits:

```bash
git commit -S --signoff
```

## Pull Requests

PR titles use the same Conventional Commits subject format.

Before opening a PR:

- run the relevant local checks where practical
- update `README.md` and `AGENTS.md` when behavior changes
- pin GitHub Actions to full commit SHA with a version comment

## Local Checks

This repository is mostly YAML, Markdown, and Renovate JSON. Recommended
checks:

```bash
yamllint .github/workflows
jq empty .github/renovate.json
```

If `lefthook` is installed, enable local hooks with:

```bash
lefthook install
```
