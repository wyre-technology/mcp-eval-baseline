# mcp-eval-baseline

Shared [mcp-assert](https://github.com/blackwell-systems/mcp-assert) baseline eval suite for the wyre-technology MCP server fleet.

## What this enforces

Every wyre-technology MCP server should pass these protocol-correctness assertions:

| Test | Asserts |
|---|---|
| `missing_credentials.yaml` | A tool call without configured credentials returns `isError: true` (not a successful response with a failure payload, and not a JSON-RPC `-32602`). |
| `unknown_tool.yaml` | A call to a non-existent tool returns an error response. |

More tests will be added as we identify additional cross-fleet protocol invariants.

## Usage

In your MCP server repo, add `.github/workflows/mcp-assert.yml`:

```yaml
name: mcp-assert

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  assert:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      - run: npm ci
      - run: npm run build --if-present
      - uses: wyre-technology/mcp-eval-baseline@main
        with:
          entry: dist/index.js          # or dist/entry.js — your build artifact
          canary-tool: foo_list_things  # any read-only tool from your server
          extra-suite: evals/           # optional: repo-local evals to run alongside
```

The action checks out this repo's `templates/`, substitutes `${ENTRY}` and `${CANARY_TOOL}`, and runs the rendered suite (plus any repo-local YAMLs in `extra-suite`).

## Why a baseline

Without a shared standard, each MCP server tests protocol behavior its own way (or not at all). A shared baseline:
- Defines the protocol contract once
- Catches divergence (e.g., one server returns `isError: true` on missing creds, another swallows the error in content text — both shipping today)
- Lets us add new assertions org-wide by editing one repo

## Adding a new baseline test

1. Add a YAML to `templates/` using `${ENTRY}` and `${CANARY_TOOL}` placeholders
2. Open a PR
3. After merge, every consuming repo picks it up on their next CI run

## Related

- [`blackwell-systems/mcp-assert`](https://github.com/blackwell-systems/mcp-assert) — the underlying tool
- [`wyre-technology/.github`](https://github.com/wyre-technology/.github) — reusable workflows
