# Code Review Guidelines

This repository is the public GitHub profile for Simon Bärlocher. It renders as
the profile landing page and holds only Markdown, JSON, and workflow YAML. There
is no application code and no deployment. Reviews focus on the rendered output
and on the security of the workflows.

## Scope

Reviews focus on correctness and security. Style preferences, formatting, and
cosmetic changes are out of scope unless they violate an explicit rule below.

## Required checks

### Rendered profile output

- The profile README renders on `github.com/sbaerlocher`. Any change to
  `README.md` must be checked against the rendered page, not just the source.
- Do not introduce the standard project sections (Overview, Quickstart,
  Configuration) — they break the profile layout. This is a documented
  profile-repo exception to the org repository standard.

### Status check name

- The aggregator job in `pull-request.yml` must keep the name
  `All Checks Passed`. The branch ruleset pins the required status check to that
  job name; renaming it silently breaks merge protection on `main`.

### Action integrity

- Every `uses:` reference must be pinned to a full 40-character commit SHA, with
  the version tag as an inline comment. Mutable tag references (`@v4`, `@main`)
  are not acceptable.
- Reusable workflows from `sbaerlocher/.github` are pinned to a date tag; that
  is the accepted form for internal reusables.

### Permissions

- Every workflow and job must declare only the permissions it actually uses.
  Omitting a top-level `permissions:` block or using `permissions: write-all`
  is a blocking issue.

### Secret handling

- Secrets must never appear in `run:` output, log statements, or environment
  variables passed to untrusted code.
- Never interpolate untrusted event input (issue/PR titles and bodies, commit
  messages, branch refs) directly into `run:` steps. Pass it through `env:` and
  quote it.

## Severity levels

| Level        | Meaning                                                        | Merge impact                  |
| ------------ | -------------------------------------------------------------- | ----------------------------- |
| Bug          | Incorrect behavior, security vulnerability, or broken contract | Blocks merge                  |
| Nit          | Minor issue — suboptimal but not incorrect                     | Non-blocking                  |
| Pre-existing | Issue present before this PR; flagged for awareness            | No action required on this PR |

## Skip

- Renovate PRs that only update SHA references and version comments
- Whitespace or formatting changes in JSON Renovate preset files
