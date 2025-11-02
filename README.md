# EcoScalpel GitHub Actions

This directory mirrors the layout of [`infracost/actions`](https://github.com/infracost/actions)
and provides reusable composite actions for the EcoScalpel ecosystem.

| Action | Description |
| --- | --- |
| [`setup`](./setup) | Builds the EcoScalpel CLI, optionally seeds credentials, and adds it to `PATH`. |
| [`run`](./run) | Generates (or reuses) a Terraform plan and executes the EcoScalpel CLI, returning report artefacts. |

## Example workflow

```yaml
name: EcoScalpel

on:
  pull_request:
    paths:
      - "**/*.tf"

jobs:
  ecoscalpel:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      pull-requests: write
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Setup Terraform
        uses: hashicorp/setup-terraform@v3
        with:
          terraform_version: 1.8.5

      - name: Setup EcoScalpel
        uses: didier-segura/actions/setup@main
        with:
          api-key: ${{ secrets.ECOSCALPEL_API_KEY }}

      - name: Run EcoScalpel
        id: run
        uses: didier-segura/actions/run@main
        with:
          setup_api_key: ${{ secrets.ECOSCALPEL_API_KEY }}
          mode: dir
          terraform_dir: terraform-scaleway-samples/samples
          format: markdown

      - name: Attach report artifact
        uses: actions/upload-artifact@v4
        with:
          name: ecoscalpel-report
          path: ${{ steps.run.outputs.report_path }}

      - name: Comment on PR
        if: github.event_name == 'pull_request'
        uses: actions/github-script@v7
        env:
          REPORT: ${{ steps.run.outputs.report }}
        with:
          script: |
            const body = `### EcoScalpel cost & sustainability report\n\n${process.env.REPORT}`;
            await github.rest.issues.createComment({
              owner: context.repo.owner,
              repo: context.repo.repo,
              issue_number: context.issue.number,
              body,
            });
```

This structure lets downstream repositories cherry-pick the pieces they need,
just like the official Infracost actions collection.
