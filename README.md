# setup-arctl

[![Latest release](https://img.shields.io/github/v/release/aairey/setup-arctl?label=action&logo=github)](https://github.com/aairey/setup-arctl/releases/latest)
[![arctl version](https://img.shields.io/github/v/release/agentregistry-dev/agentregistry?label=arctl%20default&logo=data:image/svg+xml;base64,PHN2ZyB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciIHZpZXdCb3g9IjAgMCAyNCAyNCI+PHBhdGggZmlsbD0id2hpdGUiIGQ9Ik0xMiAyQzYuNDggMiAyIDYuNDggMiAxMnM0LjQ4IDEwIDEwIDEwIDEwLTQuNDggMTAtMTBTMTcuNTIgMiAxMiAyem0tMiAxNWwtNS01IDEuNDEtMS40MUwxMCAxNC4xN2w3LjU5LTcuNTlMMTkgOGwtOSA5eiIvPjwvc3ZnPg==)](https://github.com/agentregistry-dev/agentregistry/releases/latest)
[![License](https://img.shields.io/github/license/aairey/setup-arctl)](LICENSE)
[![Platforms](https://img.shields.io/badge/platform-linux%20%7C%20macos%20%7C%20windows-blue)](#supported-platforms)

GitHub Action to install and verify the [`arctl`](https://github.com/agentregistry-dev/agentregistry) CLI from official GitHub releases.

Downloads the binary for the current OS/architecture, verifies the SHA256 checksum from the upstream `.sha256` release asset, and installs it to PATH.

## Usage

```yaml
- uses: aairey/setup-arctl@v1.0.0
```

### With a specific version

```yaml
- uses: aairey/setup-arctl@v1.0.0
  with:
    version: '0.3.3'
```

### Publish prompts

```yaml
jobs:
  publish-prompts:
    runs-on: self-hosted
    container:
      image: public.ecr.aws/docker/library/debian:bookworm-slim
    steps:
      - uses: actions/checkout@v4

      - name: Install system dependencies
        run: apt-get update && apt-get install -y --no-install-recommends curl ca-certificates

      - uses: aairey/setup-arctl@v1.0.0
        with:
          version: '0.3.3'
        env:
          ARCTL_API_BASE_URL: ${{ secrets.ARCTL_API_BASE_URL }}
          ARCTL_API_TOKEN: ${{ secrets.ARCTL_API_TOKEN }}

      - name: Publish prompts
        env:
          ARCTL_API_TOKEN: ${{ secrets.ARCTL_API_TOKEN }}
          ARCTL_API_BASE_URL: ${{ secrets.ARCTL_API_BASE_URL }}
        run: |
          find prompts -name '*.yaml' | while read -r file; do
            arctl prompt publish "$file"
          done
```

### Publish skills

```yaml
jobs:
  publish-skills:
    runs-on: self-hosted
    container:
      image: public.ecr.aws/docker/library/debian:bookworm-slim
    steps:
      - uses: actions/checkout@v4

      - name: Install system dependencies
        run: apt-get update && apt-get install -y --no-install-recommends curl ca-certificates

      - uses: aairey/setup-arctl@v1.0.0
        with:
          version: '0.3.3'
        env:
          ARCTL_API_BASE_URL: ${{ secrets.ARCTL_API_BASE_URL }}
          ARCTL_API_TOKEN: ${{ secrets.ARCTL_API_TOKEN }}

      - name: Publish skills
        env:
          ARCTL_API_TOKEN: ${{ secrets.ARCTL_API_TOKEN }}
          ARCTL_API_BASE_URL: ${{ secrets.ARCTL_API_BASE_URL }}
        run: |
          for skill_dir in skills/*/; do
            if [[ -f "$skill_dir/skill.yaml" ]]; then
              VERSION=$(grep '^version:' "$skill_dir/skill.yaml" | awk '{print $2}')
              arctl skill publish --path "$skill_dir" --version "$VERSION"
            fi
          done
```

### Deploy agent

```yaml
jobs:
  deploy-agent:
    runs-on: self-hosted
    container:
      image: public.ecr.aws/docker/library/debian:bookworm-slim
    steps:
      - uses: actions/checkout@v4

      - name: Install system dependencies
        run: apt-get update && apt-get install -y --no-install-recommends curl ca-certificates

      - uses: aairey/setup-arctl@v1.0.0
        with:
          version: '0.3.3'
        env:
          ARCTL_API_BASE_URL: ${{ secrets.ARCTL_API_BASE_URL }}
          ARCTL_API_TOKEN: ${{ secrets.ARCTL_API_TOKEN }}

      - name: Deploy agent
        env:
          ARCTL_API_TOKEN: ${{ secrets.ARCTL_API_TOKEN }}
          ARCTL_API_BASE_URL: ${{ secrets.ARCTL_API_BASE_URL }}
        run: |
          VERSION=$(grep '^version:' agent.yaml | awk '{print $2}')
          arctl agent deploy --version "$VERSION" --namespace "${{ vars.DEPLOY_NAMESPACE }}"
```

> **Connectivity:** Two environment variables are required to reach your agentregistry instance:
>
> | Variable | Description | Example |
> |----------|-------------|---------|
> | `ARCTL_API_BASE_URL` | Full base URL of the registry (include `https://`) | `https://agentregistry.example.com` |
> | `ARCTL_API_TOKEN` | Bearer token for authentication | *(from your identity provider)* |
>
> Both are read automatically by `arctl` — no `--registry-url` or `--registry-token` flags needed.
> Store them as secrets and pass via `env:` in your job steps.

## Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `version` | arctl version to install (without leading `v`) | No | `0.3.3` |
| `install-dir` | Directory to install arctl into | No | `/usr/local/bin` |

## How it works

1. Detects the current OS and architecture (`linux/darwin` × `amd64/arm64`)
2. Downloads the matching `arctl` binary from the GitHub release
3. Fetches the corresponding `.sha256` file from the same release
4. Verifies SHA256 checksum — fails the step if it doesn't match
5. Installs the verified binary to `install-dir`

## Supported platforms

| OS | Architecture |
|----|-------------|
| Linux | amd64, arm64 |
| macOS | amd64, arm64 |
| Windows | amd64 |

## License

[Apache 2.0](LICENSE) — see [NOTICE](NOTICE) for upstream attribution.
