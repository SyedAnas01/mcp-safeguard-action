# MCP Safeguard Scan

GitHub Action that runs [mcp-safeguard](https://github.com/SyedAnas01/mcp-safeguard) on an MCP server repository or an MCP client config, and uploads the results to GitHub Code Scanning.

It checks for:

- shell injection (tool input reaching `subprocess`/`os.system`/`child_process.exec`)
- SSRF and unvalidated outbound fetches
- hardcoded credentials (including OpenAI `sk-proj-` keys and per-server `env` blocks)
- disabled TLS verification
- prompt injection and tool poisoning in tool descriptions

## Usage

```yaml
name: MCP security scan
on: [push, pull_request]

permissions:
  contents: read
  security-events: write   # needed for the SARIF upload

jobs:
  mcp-safeguard:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: SyedAnas01/mcp-safeguard-action@v1
        with:
          mode: scan-source
          path: .
          fail-on: HIGH
```

Findings show up under **Security > Code scanning** and as annotations on pull requests.

Scan a client config instead of server source:

```yaml
      - uses: SyedAnas01/mcp-safeguard-action@v1
        with:
          mode: scan
          path: .mcp.json
          fail-on: CRITICAL
```

Report only, never fail the build:

```yaml
      - uses: SyedAnas01/mcp-safeguard-action@v1
        with:
          continue-on-error: 'true'
```

## Badge

Each run writes a job summary with finding counts and a ready-to-paste badge for your README:

```markdown
[![MCP security](https://github.com/OWNER/REPO/actions/workflows/mcp-safeguard.yml/badge.svg)](https://github.com/OWNER/REPO/actions/workflows/mcp-safeguard.yml)
[![scanned by mcp-safeguard](https://img.shields.io/badge/scanned%20by-mcp--safeguard-2b6cb0)](https://github.com/SyedAnas01/mcp-safeguard)
```

The first badge is live: it shows whether your latest scan passed.

## Inputs

| Input | Default | Meaning |
|---|---|---|
| `mode` | `scan-source` | `scan-source` walks a source tree; `scan` reads an MCP config/tool JSON |
| `path` | `.` | Directory (scan-source) or JSON file (scan) |
| `fail-on` | `HIGH` | Fail the job on findings at or above this severity |
| `severity` | `INFO` | Minimum severity to report |
| `format` | `sarif` | `sarif`, `json` or `text` |
| `output` | `mcp-safeguard-results.sarif` | Report file path |
| `upload-sarif` | `true` | Upload to Code Scanning when format is sarif |
| `version` | latest | Pin a PyPI version of mcp-safeguard, e.g. `0.10.2` |
| `python-version` | `3.12` | Python used on the runner |
| `continue-on-error` | `false` | Report findings without failing the job |

## Outputs

| Output | Meaning |
|---|---|
| `report-path` | Path of the written report |
| `exit-code` | Scanner exit code (0 = nothing at or above `fail-on`) |

## pre-commit

The same checks run locally before each commit:

```yaml
repos:
  - repo: https://github.com/SyedAnas01/mcp-safeguard
    rev: v0.10.2
    hooks:
      - id: mcp-safeguard          # MCP config files
      - id: mcp-safeguard-source   # MCP server source tree
```

## False positives

Open an issue on [mcp-safeguard](https://github.com/SyedAnas01/mcp-safeguard/issues) with the rule ID and a minimal snippet.

## License

MIT
