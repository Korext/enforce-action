# Korext Enforce Action

[![GitHub Marketplace](https://img.shields.io/badge/Marketplace-Korext%20Enforce-orange?logo=github)](https://github.com/marketplace/actions/korext-enforce)
[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)

Enforce security, compliance, and quality standards on AI-generated code directly in your GitHub workflows. Korext scans your codebase against policy packs and surfaces violations as GitHub Code Scanning annotations on pull requests.

## Quick Start

Add this to `.github/workflows/korext.yml`:

```yaml
name: Korext Enforcement
on: [push, pull_request]

permissions:
  contents: read
  security-events: write

jobs:
  enforce:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: Korext/enforce-action@v1
```

That's it. Korext scans your code on every push and PR using the default `web` policy pack.

## How It Works

1. **Install** -- The action installs the [Korext CLI](https://www.npmjs.com/package/korext) (`v0.9.5`)
2. **Scan** -- Runs `korext enforce` against your codebase with the selected policy pack
3. **Report** -- Generates a SARIF file and uploads it to GitHub Code Scanning
4. **Gate** -- Fails the workflow if critical or high severity violations are found

Violations appear as annotations directly on the PR diff, powered by GitHub Code Scanning.

## Inputs

| Input | Description | Required | Default |
|---|---|---|---|
| `directory` | Directory to scan for policy violations | No | `.` |
| `pack` | Policy Pack ID to enforce | No | `web` |
| `api-token` | Korext API token for authenticated mode | No | _(anonymous)_ |
| `fail-on-violations` | Fail workflow on critical/high violations | No | `true` |
| `sarif-upload` | Upload SARIF to GitHub Code Scanning | No | `true` |

## Outputs

| Output | Description |
|---|---|
| `violations` | Total number of policy violations found |
| `sarif-file` | Path to the generated SARIF results file |

## Examples

### Basic usage (anonymous mode)

```yaml
- uses: actions/checkout@v4
- uses: Korext/enforce-action@v1
```

### With authentication

```yaml
- uses: actions/checkout@v4
- uses: Korext/enforce-action@v1
  with:
    api-token: ${{ secrets.KOREXT_API_TOKEN }}
```

### Custom policy pack

```yaml
- uses: actions/checkout@v4
- uses: Korext/enforce-action@v1
  with:
    pack: owasp-top-10
```

### Scan a subdirectory

```yaml
- uses: actions/checkout@v4
- uses: Korext/enforce-action@v1
  with:
    directory: src/
```

### Warn but don't fail

```yaml
- uses: actions/checkout@v4
- uses: Korext/enforce-action@v1
  with:
    fail-on-violations: 'false'
```

### PR-only enforcement

```yaml
name: Korext PR Check
on: pull_request

permissions:
  contents: read
  security-events: write

jobs:
  enforce:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: Korext/enforce-action@v1
        with:
          pack: web
```

### Use violation count in downstream steps

```yaml
- uses: actions/checkout@v4
- uses: Korext/enforce-action@v1
  id: korext
  with:
    fail-on-violations: 'false'
- run: echo "Found ${{ steps.korext.outputs.violations }} violations"
```

## Permissions

Your workflow must include:

```yaml
permissions:
  contents: read          # Checkout repository
  security-events: write  # Upload SARIF to Code Scanning
```

Without `security-events: write`, SARIF upload will be skipped.

## Rate Limits

| Mode | Limit | How to use |
|---|---|---|
| Anonymous (no token) | 20 requests per hour per IP | Default, no setup needed |
| Authenticated | 500+ requests per period | Set `api-token` input |

Each workflow run consumes 1 request (one `korext enforce` call). Anonymous mode is sufficient for most open-source projects. For high-traffic repos with many PRs, use an API token.

## Code Scanning

When `sarif-upload` is `true` (the default), the action uploads SARIF results to GitHub Code Scanning. Violations appear as annotations on the PR diff with severity levels:

- **Error** -- Critical and high severity violations
- **Warning** -- Medium severity violations
- **Note** -- Low severity violations

> **Note:** GitHub Code Scanning is free for public repositories. Private repositories require [GitHub Advanced Security](https://docs.github.com/en/get-started/learning-about-github/about-github-advanced-security).

## Supported File Types

Korext scans the following file types:

`.ts` `.tsx` `.js` `.jsx` `.py` `.go` `.java` `.rs`

The scanner automatically skips `node_modules`, `.git`, `dist`, `build`, and `.next` directories.

## Exit Codes

| Exit Code | Meaning | Action Behavior |
|---|---|---|
| 0 | No violations, or only medium/low severity | Workflow passes |
| 1 | Critical or high severity violations found | Workflow fails (if `fail-on-violations` is `true`) |
| 2 | Errors analyzing some files | Workflow fails |

## Links

- [Korext Platform](https://app.korext.com)
- [Korext CLI on npm](https://www.npmjs.com/package/korext)
- [Korext Website](https://www.korext.com)
- [Korext Docs] (https://www.korext.com/docs)

## License

[MIT](LICENSE)
