# EcoScalpel Run Action

This composite action orchestrates a Terraform plan (optional) and invokes the
[EcoScalpel](../../ecoscalpel) CLI to produce Scaleway cost, CO₂, and water
estimates. It mirrors the [`infracost/actions`](https://github.com/infracost/actions/)
run action and can be combined with additional steps (e.g., PR comments or
artifact uploads) to form a complete pipeline.

## Usage

```yaml
jobs:
  ecoscalpel:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      pull-requests: write
    steps:
      - name: Check out repository
        uses: actions/checkout@v4

      - name: Set up Terraform
        uses: hashicorp/setup-terraform@v3
        with:
          terraform_version: 1.8.5

      - name: Run EcoScalpel
        id: run
        uses: didier-segura/actions/run@main
        with:
          setup_api_key: ${{ secrets.ECOSCALPEL_API_KEY }}
          mode: dir
          terraform_dir: terraform-scaleway-samples/samples
          format: markdown

      - name: Upload EcoScalpel report
        uses: actions/upload-artifact@v4
        with:
          name: ecoscalpel-report
          path: ${{ steps.run.outputs.report_path }}

      - name: Comment report on PR
        if: github.event_name == 'pull_request'
        uses: actions/github-script@v7
        env:
          REPORT: ${{ steps.run.outputs.report }}
        with:
          script: |
            const body = `### EcoScalpel cost & sustainability report\n\n${process.env.REPORT}`;
            github.rest.issues.createComment({
              owner: context.repo.owner,
              repo: context.repo.repo,
              issue_number: context.issue.number,
              body,
            });
```

### Inputs

| Input | Description | Default |
| --- | --- | --- |
| `mode` | How EcoScalpel consumes input. `plan` uses Terraform plan JSON, `dir` reads source files directly. | `plan` |
| `terraform_dir` | Directory that holds the Terraform configuration. | `.` |
| `plan_file` | Path (relative to the repo) where the plan JSON is stored/read. | `ecoscalpel-plan.json` |
| `skip_plan` | Set to `true` when you already generated the plan JSON. | `false` |
| `terraform_plan_args` | Extra flags appended to `terraform plan`. | `-lock=false` |
| `format` | EcoScalpel output format (`table`, `markdown`, `json`). | `markdown` |
| `style` | `-style` flag passed to EcoScalpel (leave blank to suppress). | `fancy` |
| `output_file` | Where to write the CLI output. Inferred from `format` if omitted. | |
| `push_endpoint`, `push_api_key`, `push_project`, `push_metadata`, `push_fail_on_error` | Forwarded to EcoScalpel for collector uploads. | |
| `setup_api_key` | API key forwarded to the setup action (stored in the CLI config). | |
| `setup_token` | GitHub token forwarded to the setup action for downloading private releases. | |
| `download_url` | Direct download URL passed to the setup action. | |
| `version` | EcoScalpel release version to install (default `latest`). | |

### Outputs

| Output | Description |
| --- | --- |
| `plan_path` | Absolute path to the plan JSON analysed by EcoScalpel. |
| `report_path` | Absolute path to the saved report file. |
| `report` | Raw EcoScalpel output (matches `format`). |

Combine this action with [`actions/setup`](../setup) when you want direct
control over the CLI or to share the compiled binary across multiple jobs.
