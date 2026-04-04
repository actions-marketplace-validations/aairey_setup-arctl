# setup-arctl

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

### Full example — publish prompts to agentregistry

```yaml
jobs:
  publish:
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

      - name: Publish prompts
        env:
          ARCTL_API_TOKEN: ${{ secrets.ARCTL_API_TOKEN }}
          ARCTL_API_BASE_URL: ${{ secrets.AGENTREGISTRY_URL }}
        run: |
          find prompts -name '*.yaml' | while read -r file; do
            arctl prompt publish "$file"
          done
```

> **Note:** `ARCTL_API_TOKEN` is the bearer token for your agentregistry instance.
> `ARCTL_API_BASE_URL` is the base URL of your registry (e.g. `https://agentregistry.example.com`).
> Both are read automatically by `arctl` — no `--registry-url` or `--registry-token` flags needed.

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
