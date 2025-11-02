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
| `install_dir` | Directory that will receive the `ecoscalpel` binary. | `${{ runner.temp }}/ecoscalpel/bin` |
| `api-key` | Optional API key written to `~/.config/ecoscalpel/config.json`. | |
| `download_url` | Direct download URL for the EcoScalpel binary. Overrides the version setting. | |
| `version` | Release tag used to construct the download URL when `download_url` is blank (e.g. `prod-0.0.7`). | _required if `download_url` omitted_ |
| `token` | GitHub token included in the download request (needed for private releases). | |

### Outputs

| Output | Description |
| --- | --- |
| `ecoscalpel-path` | Absolute path to the compiled EcoScalpel binary. |

> **Note**
> When `download_url` is not supplied, the action downloads
> `https://github.com/didier-segura/ecoscalpel/releases/download/<version>/ecoscalpel-<os>-<arch>`.
> Provide `token` (for example `${{ secrets.GITHUB_TOKEN }}` or a PAT) when the release lives in a private repository.

You can combine this setup action with other custom actions (for example,
[`actions/run`](../run)) or call the CLI directly in subsequent steps.
