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
- Can optionally fail the job on a failed audit through `fail_on_audit: true`.
- Fails the job only when the workflow cannot execute the scan correctly.
- By default, does not fail the job only because Kiuwan reports findings or a failed audit.

Required secrets:

- `VPN_CONFIG_B64`: Base64-encoded contents of the OpenVPN client configuration file (`.ovpn`).
- `VPN_USER`: OpenVPN username.
- `VPN_PASS`: OpenVPN password.
- `KIUWAN_USER`: Kiuwan username.
- `KIUWAN_PASS`: Kiuwan password.
