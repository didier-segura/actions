# Setup EcoScalpel Action

This composite action installs the [EcoScalpel](https://github.com/didier-segura/ecoscalpel)
CLI so other GitHub workflows can run cost and sustainability checks against
Scaleway Terraform projects. The behaviour mirrors the
[`infracost/actions`](https://github.com/infracost/actions/) setup action: it
downloads the published release binary, places it on `PATH`, and optionally
stores an API key for collector uploads.

## Usage

```yaml
jobs:
  prepare:
    runs-on: ubuntu-latest
    steps:
      - name: Check out repository
        uses: actions/checkout@v4

      - name: Install EcoScalpel
        id: setup
        uses: scaleway/ecoscalpel/actions/setup@main
        with:
          api-key: ${{ secrets.ECOSCALPEL_API_KEY }}

      - name: Show version
        run: ${{ steps.setup.outputs.ecoscalpel-path }} --help
```

### Inputs

| Input | Description | Default |
| --- | --- | --- |
| `install-dir` | Directory that will receive the `ecoscalpel` binary. | `${{ runner.temp }}/ecoscalpel/bin` |
| `api-key` | Optional API key written to `~/.config/ecoscalpel/config.json`. | |
| `version` | Release tag to install (e.g. `v0.8.1`). Use `latest` for the most recent release. | `latest` |

### Outputs

| Output | Description |
| --- | --- |
| `ecoscalpel-path` | Absolute path to the compiled EcoScalpel binary. |

> **Note**
> The action expects EcoScalpel releases to publish assets named
> `ecoscalpel_<os>_<arch>.tar.gz` (for example `ecoscalpel_linux_amd64.tar.gz`).
> The archive must contain a single `ecoscalpel` binary.

You can combine this setup action with other custom actions (for example,
[`actions/run`](../run)) or call the CLI directly in subsequent steps.
