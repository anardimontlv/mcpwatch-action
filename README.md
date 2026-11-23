# mcpwatch GitHub Action

Know when the tools your AI agents trust change. This Action runs [mcpwatch](https://github.com/anardimontlv/mcpwatch)
in your CI:

- **scan**: flags risky MCP configs (leaked secrets, unpinned packages, plain HTTP, root filesystem access,
  privileged containers, tool poisoning).
- **check**: compares your MCP servers against your committed `mcpwatch.lock` and fails when a pinned tool's
  description or schema changes (a "rug pull"), when an endpoint drops auth, or when fingerprints go stale.

Static and read-only: it never runs third-party server code, and your config never leaves the runner.

## Scan on every pull request

```yaml
on: pull_request
jobs:
  mcp-scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: anardimontlv/mcpwatch-action@v1
        with: { config: .mcp.json, fail-on: high }
```

## Check for drift every day

Create the lock file once (`pipx install mcpwatch-io && mcpwatch lock .mcp.json`),
commit `mcpwatch.lock`, then:

```yaml
on: { schedule: [{ cron: "0 6 * * *" }] }
jobs:
  mcp-drift:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: anardimontlv/mcpwatch-action@v1
        with: { mode: check, config: .mcp.json, lock: mcpwatch.lock }
```

Exit codes: 0 OK, 1 findings at or above `fail-on` (scan) or drift (check), 2 bad input or index unreachable.

Docs, rules and methodology: https://github.com/anardimontlv/mcpwatch · Index and radar: https://www.mcpwatch.io

Apache-2.0.

## Team: org view and policy

Report each repo's MCP servers to your mcpwatch Team dashboard and fail the job on policy violations
(no plain HTTP, pinned packages, minimum grade, ...). Only public server ids and rule codes are sent:
never your config contents, server names, URLs or secrets. Create the key on https://www.mcpwatch.io/watch.

```yaml
- uses: anardimontlv/mcpwatch-action@v1
  with:
    mode: report
    report-key: ${{ secrets.MCPWATCH_REPORT_KEY }}
```
