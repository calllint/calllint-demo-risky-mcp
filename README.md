# calllint-demo-risky-mcp

A deliberately mixed MCP configuration that demonstrates [CallLint](https://calllint.com)
running in CI and publishing results to **GitHub Code Scanning** — **see
CallLint fail a risky PR without ever executing an MCP server.**

CallLint is a deterministic, offline, static **pre-flight** scanner for MCP and
agent-tool configurations. Before your agent acts, it checks the blast radius:
what each tool can read, write, execute, connect to, send, or mutate — and
returns evidence-backed `SAFE` / `REVIEW` / `BLOCK` / `UNKNOWN` verdicts. It
**never executes, installs, or connects to** the servers it judges — including
the ones in this repo.

> This is a demo. The config below is intentionally risky to exercise every
> verdict. Do not copy it into a real project.

## What's here

```text
.cursor/mcp.json              4 servers, one per verdict
.github/workflows/calllint.yml  scan → SARIF → Code Scanning + HTML artifact
reports/expected.md           the verdict each server should get, and why
```

## The four servers

| Server | Verdict | Why |
|--------|---------|-----|
| `safe-local-project` | **SAFE** | Pinned docker image, container-scoped db path, no secrets, no shell. |
| `review-unpinned-npx` | **REVIEW** | Runs an unpinned npx package — review-worthy, not blocking. |
| `block-broad-filesystem` | **BLOCK** | Grants filesystem access to a broad home path (`/Users/username`). |
| `unknown-remote-vendor` | **UNKNOWN** | Points at an unverified remote endpoint that can't be inspected statically. |

Overall verdict for the config is **BLOCK** (the worst server wins).

## See it in CI

1. Open the **Actions** tab → the `CallLint` workflow run.
2. Open the **Security → Code scanning** tab → CallLint alerts, one per finding.
3. Download the **`calllint-html-report`** artifact from the workflow run for the
   full self-contained HTML report.

The workflow uploads the SARIF and HTML *before* the policy gate. The gate step
runs `--ci` (which exits 30 on `BLOCK`) but is **report-only here**
(`continue-on-error: true`) so this demo's run stays green and the risk signal
shows up as Code Scanning alerts. In your own repo, delete `continue-on-error`
to fail PRs on `BLOCK` / `UNKNOWN`.

## Run it yourself

```bash
npx calllint scan .cursor/mcp.json            # human-readable
npx calllint scan .cursor/mcp.json --sarif    # SARIF 2.1.0 to stdout
npx calllint scan .cursor/mcp.json --html     # HTML report to stdout
npx calllint scan .cursor/mcp.json --ci       # exit non-zero on BLOCK/UNKNOWN
```

The commands above install the latest stable `calllint` from npm (the `latest`
dist-tag, currently the `0.3.x` line). Release candidates are on `@next`; older
previews on `@preview`. Use stable by default.

## What this demo proves — and does not

This demo proves:

- CallLint can classify representative config surfaces (SAFE / REVIEW / BLOCK /
  UNKNOWN) from a real `.cursor/mcp.json`.
- SARIF can be uploaded to GitHub Code Scanning, one alert per finding.
- No target MCP server needs to run — CallLint is a static, offline pre-flight
  check.

This demo does **not** prove:

- Runtime safety of any MCP server.
- Complete MCP ecosystem coverage.
- That the third-party tools referenced are benign.

CallLint is heuristic decision support, not a safety guarantee. A clean run is
necessary, not sufficient. See the
[main project](https://github.com/calllint/calllint) for the threat model,
[`LIMITATIONS.md`](https://github.com/calllint/calllint/blob/main/LIMITATIONS.md)
for the trust boundaries, and
[`docs/project-facts.json`](https://github.com/calllint/calllint/blob/main/docs/project-facts.json)
for the current corpus numbers.

## Limitations

CallLint scans configuration, not runtime behavior. It does not prove a server
is safe — only that its declared configuration does or doesn't trip known risk
heuristics. It is pre-1.0; expect false positives and false negatives. See the
[main project](https://github.com/calllint/calllint) for the threat model.
