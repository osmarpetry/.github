# .github

Shared workflows for all [@osmarpetry](https://github.com/osmarpetry) repos.

One policy, one place: a dependency opens a PR, CI runs, green squash-merges itself.

## Usage (this org)

This repo is public, so any repo can call it directly. Add `.github/workflows/ci.yml`
to the consumer repo:

```yaml
name: ci

on:
  pull_request:
  push:
    branches: [main]

concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true

jobs:
  build:
    uses: osmarpetry/.github/.github/workflows/node.yml@main

  dependabot:
    needs: [build]
    if: github.actor == 'dependabot[bot]'
    permissions:
      contents: write
      pull-requests: write
    uses: osmarpetry/.github/.github/workflows/dependabot-automerge.yml@main
```

`needs: [build]` is what makes automerge wait for CI — never call `dependabot-automerge.yml` without it.

## Workflows

| Workflow | What it does |
| --- | --- |
| `node.yml` | Detects pnpm/bun/yarn/npm from the lockfile and the Node version from `.nvmrc`; installs with a frozen lockfile and runs `lint`, `typecheck`/`type-check`, `test:ci` or `test`, `build` — each only if the script exists. |
| `go.yml` | `go build ./...` + `go test ./...` |
| `maven.yml` | `mvn -B verify` |
| `dependabot-automerge.yml` | Squash-merges a `dependabot[bot]` PR. Always call it with `needs:` on the build job. |

Third-party actions are pinned by full SHA, tag in a comment. Dependabot bumps the SHAs weekly.

## Using this in a private org (e.g. a company repo)

Same workflows, different host repo — you can't call `osmarpetry/.github` from
somewhere it has no business being called from.

1. Copy `.github/workflows/*.yml` into a shared repo in that org, e.g. `<org>/.github`.
2. If that repo is **private**: repo Settings → Actions → General → *Access*, select
   "Accessible from repositories in the '\<org\>' organization", Save. Public repos
   need no such step.
3. Point consumer repos' `uses:` at `<org>/.github/.github/workflows/node.yml@main`
   instead of `osmarpetry/...`.

No `secrets: inherit` needed — the only secret used is the default `GITHUB_TOKEN`,
scoped per job via `permissions:`.
