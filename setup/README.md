# Setup EcoScalpel Action

This composite action installs the [EcoScalpel](../../ecoscalpel) CLI so other
GitHub workflows can run cost and sustainability checks against Scaleway
Terraform projects. The behaviour mirrors the [`infracost/actions`](https://github.com/infracost/actions/)
setup action: it builds the CLI from source and adds it to `PATH`, ready for
later steps.

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
| `repo` | Git repository URL cloned to build the CLI. | `https://github.com/didier-segura/ecoscalpel.git` |
| `ref` | Git ref (branch, tag, commit) to checkout before building. | `main` |

### Outputs

| Output | Description |
| --- | --- |
| `ecoscalpel-path` | Absolute path to the compiled EcoScalpel binary. |

You can combine this setup action with other custom actions (for example,
[`actions/run`](../run)) or call the CLI directly in subsequent
steps.
