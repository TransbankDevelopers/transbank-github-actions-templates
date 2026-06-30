# Transbank GitHub Actions Templates

Reusable workflows shared across repositories.

## Kiuwan PR Scan

Workflow path:

`/.github/workflows/kiuwan-pr-scan.yml`

Expected caller usage:

```yaml
name: Kiuwan

on:
  pull_request:

jobs:
  scan:
    uses: transbankdevelopers/transbank-github-actions-templates/.github/workflows/kiuwan-pr-scan.yml@main
    with:
      project_name: td-pos-sdk-dotnet
      source_path: .
      fail_on_audit: true
    secrets:
      VPN_CONFIG_B64: ${{ secrets.VPN_CONFIG_B64 }}
      VPN_USER: ${{ secrets.VPN_USER }}
      VPN_PASS: ${{ secrets.VPN_PASS }}
      KIUWAN_USER: ${{ secrets.KIUWAN_USER }}
      KIUWAN_PASS: ${{ secrets.KIUWAN_PASS }}
```

Behavior:

- Connects to the corporate network through OpenVPN.
- Downloads Kiuwan Local Analyzer at runtime.
- Executes the PR scan from CLI without project dependencies.
- Publishes the scan result in the step summary and as a fresh PR comment, removing the previous Kiuwan comment first.
- Reports failed audits explicitly as `Failed` in the summary/comment.
- Can optionally fail the job unless the audit result is explicitly `Passed` through `fail_on_audit: true`.
- Fails the job only when the workflow cannot execute the scan correctly.
- By default, does not fail the job only because Kiuwan reports findings or a failed audit.

Required secrets:

- `VPN_CONFIG_B64`: Base64-encoded contents of the OpenVPN client configuration file (`.ovpn`).
- `VPN_USER`: OpenVPN username.
- `VPN_PASS`: OpenVPN password.
- `KIUWAN_USER`: Kiuwan username.
- `KIUWAN_PASS`: Kiuwan password.

## Comment Plugin Artifact Link on PR

Workflow path:

`/.github/workflows/plugin-pr-artifact.yml`

Expected caller usage:

```yaml
name: Comment Plugin Artifact Link on PR

on:
  pull_request:

jobs:
  build:
    runs-on: ubuntu-latest
    outputs:
      artifact_url: ${{ steps.artifact_upload.outputs.artifact-url }}
    steps:
      - uses: actions/checkout@v4

      - name: Build plugin ZIP
        run: npm run build

      - name: Upload build artifact
        id: artifact_upload
        uses: actions/upload-artifact@v4
        with:
          name: transbank-plugin-prestashop-webpay-rest.zip
          path: build/transbank-plugin-prestashop-webpay-rest.zip
          if-no-files-found: error

  comment:
    needs: build
    uses: transbankdevelopers/transbank-github-actions-templates/.github/workflows/plugin-pr-artifact.yml@main
    with:
      pr_number: ${{ github.event.pull_request.number }}
      source_repo_full_name: ${{ github.event.pull_request.head.repo.full_name }}
      artifact_url: ${{ needs.build.outputs.artifact_url }}
      artifact_name: transbank-plugin-prestashop-webpay-rest.zip
      comment_prefix: Build listo
```

Behavior:

- The comment job runs after the build job succeeds.
- The consumer workflow uploads the ZIP artifact.
- Comments on the PR with the artifact link when the PR comes from the same repository.
- Reuses a hidden marker so each new run updates the existing comment instead of creating duplicates.
- Cancels older in-flight comment jobs for the same PR so the last successful execution wins.

Inputs:

- `pr_number`: PR number to comment on.
- `source_repo_full_name`: Source repo full name from the pull request payload.
- `artifact_url`: URL returned by `actions/upload-artifact`.
- `artifact_name`: Display name for the uploaded artifact and the PR comment link.
- `comment_marker`: Hidden marker used to find and update the existing PR comment.
- `comment_prefix`: Visible text shown before the artifact link.
