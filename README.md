# .github

Workflows compartilhados de todos os repos de [@osmarpetry](https://github.com/osmarpetry).

Política única, um lugar só: dependência abre PR, CI roda, verde faz squash-merge sozinho.

## Uso

`.github/workflows/ci.yml` no repo consumidor:

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

## Workflows

| Workflow | O que faz |
| --- | --- |
| `node.yml` | Detecta pnpm/bun/yarn/npm pelo lockfile e a versão do Node pelo `.nvmrc`; instala com lockfile congelado e roda `lint`, `typecheck`/`type-check`, `test:ci` ou `test`, `build` — cada um só se o script existir. |
| `go.yml` | `go build ./...` + `go test ./...` |
| `maven.yml` | `mvn -B verify` |
| `dependabot-automerge.yml` | Aprova e agenda squash-merge de PR do `dependabot[bot]`. Sempre com `needs:` no build. |

Actions de terceiros são fixadas por SHA completo, com a tag em comentário. O Dependabot atualiza os SHAs semanalmente.
