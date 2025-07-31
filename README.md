<!--
SPDX-FileCopyrightText: 2025 DB Systel GmbH

SPDX-License-Identifier: Apache-2.0
-->

# Web Deployment Action

<!-- action-docs-description source="action.yml" -->
## Description

Deploys a built website artifact to a production and preview environment via SSH, with optional link checking.
<!-- action-docs-description source="action.yml" -->

- Downloads the specified artifact
- Deploys it via SSH/SCP to a production and preview directory
- Optionally runs a link checker (via [Lychee](https://github.com/lycheeverse/lychee))
- Can post/update sticky PR comments and GitHub step summaries (if enabled)

You can define which conditions have to be met in order to start a productive deployment. Preview deployments are made for all pull requests.

## Usage

```yaml
name: Preview and deploy Website

on:
  pull_request:
    types: [opened, reopened, synchronize, closed]
  push:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-24.04
    steps:
      - uses: actions/checkout@v4
      # Insert here your website build process, below a boilerplate
      - name: Build Website
        run: |
          mkdir -p dist && echo "<html>Hello</html>" > dist/index.html
      # Artifact has to be uploaded, using `artifact_name` as name
      - uses: actions/upload-artifact@v4
        with:
          name: website
          path: dist

  deploy:
    needs: build
    uses: openrail/web-deployment-action@v1
    with:
      artifact_name: website
      domain_production: example.com
      domain_preview: preview.example.com
      ssh_host: ssh.example.com
      ssh_user: webmaster
      dir_base: /var/www/
      dir_production: html
      dir_preview_base: preview
      dir_preview_subdir: pr-${{ github.event.number }}
    secrets:
      ssh_key: ${{ secrets.SSH_PRIVATE_KEY }}
```

## Requirements

- A built website must be uploaded as an artifact in a previous job.
- A valid SSH Private Key must be provided via repository or organization secrets. SSH passwords are not supported.

<!-- action-docs-inputs source="action.yml" -->
## Inputs

| name | description | required | default |
| --- | --- | --- | --- |
| `artifact_name` | <p>The name of the uploaded artifact to deploy.</p> | `true` | `""` |
| `domain_production` | <p>Domain name for the production deployment, e.g. https://example.com</p> | `false` | `""` |
| `domain_preview` | <p>Domain name for the preview deployment, e.g. https://preview.example.com</p> | `true` | `""` |
| `ssh_host` | <p>SSH hostname (e.g. webspace.example.org)</p> | `true` | `""` |
| `ssh_user` | <p>SSH username</p> | `true` | `""` |
| `ssh_port` | <p>SSH port</p> | `false` | `22` |
| `ssh_timeout` | <p>SSH connection timeout (e.g. 1m)</p> | `false` | `1m` |
| `ssh_command_timeout` | <p>SSH command timeout (e.g. 2m)</p> | `false` | `2m` |
| `ssh_rm` | <p>Remove remote directory before deploying (true/false)</p> | `false` | `true` |
| `ssh_strip_components` | <p>How many path components to strip (for tar/scp unpacking)</p> | `false` | `1` |
| `dir_base` | <p>Remote base directory (e.g. /var/www/virtual)</p> | `true` | `""` |
| `dir_production` | <p>Full path for production deployment</p> | `true` | `""` |
| `dir_preview_base` | <p>Preview base domain (e.g. preview)</p> | `true` | `""` |
| `dir_preview_subdir` | <p>Preview subdir (e.g. pr-123)</p> | `true` | `""` |
| `linkchecker_enabled` | <p>Enable link checker (true/false)</p> | `false` | `true` |
| `linkchecker_exclude` | <p>Comma-separated list of domains to exclude from link checker</p> | `false` | `""` |
| `linkchecker_include_fragments` | <p>Include anchor fragments in link checking</p> | `false` | `true` |
| `linkchecker_max_concurrency` | <p>Max concurrent requests for link checker</p> | `false` | `1` |
| `linkchecker_user_agent` | <p>User-Agent for link checking</p> | `false` | `Mozilla/5.0 (Windows NT 10.0; Win64; x64)...` |
| `linkchecker_retry_times` | <p>Retry delay in seconds for link checking</p> | `false` | `5` |
| `linkchecker_fail_on_errors` | <p>Fail workflow on link check errors (true/false)</p> | `false` | `false` |
| `sticky_comment_enabled` | <p>Whether to enable sticky comments for the pull request. Defaults to true.</p> | `false` | `true` |
| `step_summary_enabled` | <p>Whether to enable step summaries in the GitHub Actions UI. Defaults to true.</p> | `false` | `true` |
<!-- action-docs-inputs source="action.yml" -->

<!-- Note: Secrets filled in manually -->
## Secrets

| name | description | required | default |
| --- | --- | --- | --- |
| `ssh_key` | <p>SSH private key for authenticating with the deployment server.</p> | `true` | `""` |


<!-- action-docs-outputs source="action.yml" -->
## Outputs

| name | description |
| --- | --- |
| `production_url` | <p>The URL of the deployed production website</p> |
| `preview_url` | <p>The URL of the deployed preview website</p> |
<!-- action-docs-outputs source="action.yml" -->

<!-- action-docs-runs source="action.yml" -->
## Runs

This action is a `workflow` action.
<!-- action-docs-runs source="action.yml" -->

## Development and Contribution

We welcome contributions to improve this action. Please read [CONTRIBUTING.md](./CONTRIBUTING.md) for all information.

## License

The content of this repository is licensed under the [Apache 2.0 license](https://www.apache.org/licenses/LICENSE-2.0).

There may be components under different, but compatible licenses or from different copyright holders. The project is REUSE compliant which makes these portions transparent. You will find all used licenses in the [LICENSES](./LICENSES/) directory.

The project has been started by the [OpenRail Association](https://openrailassociation.org). You are welcome to [contribute](./CONTRIBUTING.md)!
