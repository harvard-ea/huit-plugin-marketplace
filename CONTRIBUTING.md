# Contributing

## Use the skill

Install `manage-marketplace` from this marketplace and ask Claude to add, update,
deprecate, or remove an entry. It writes every file, validates against
`schema/`, and pushes or opens a PR depending on this repo's branch protection.

```
/plugin marketplace add harvard-ea/huit-plugin-marketplace
/plugin install manage-marketplace@huit-plugin-marketplace
```

Publishing requires **write access to this repo** — that is the whole access
control story. No Okta, no bot, no allowlist.

## By hand

Hand-authored PRs are fine; there is no generator to run. An entry is a
directory under `servers/` (MCP server) or `plugins/` (skills only) containing:

- `server.yaml` — the source of truth
- `.claude-plugin/plugin.json` and `.codex-plugin/plugin.json`
- `.mcp.json` — MCP servers only
- `.codex-plugin/mcp.json` — credentialed servers only (Codex reads
  `http_headers`, not Claude's `headers`)
- `README.md`

…plus an entry in **both** catalogs. Validate before pushing:

```bash
npx --yes ajv-cli@5 validate --spec=draft7 \
  -s schema/claude-marketplace.schema.json -d .claude-plugin/marketplace.json
```

`ajv` exits non-zero both for invalid data and for an invalid schema — read the
message, not just the exit code.

## Two rules that bite

- **Never put `url` on a catalog plugin entry.** Claude Code's loader rejects
  entries with unrecognized keys; this specific mistake broke installs once. The
  endpoint belongs in `server.yaml` and `.mcp.json`.
- **Update both catalogs in the same commit.** Changing one leaves the repo
  half-broken for the other ecosystem.

## CI

`.github/workflows/validate.yml` validates catalogs and every entry against
`schema/`, and checks each catalog entry points at a directory that exists. It
requires **no repository secret**.

`.github/workflows/health.yml` is an opt-in weekly probe of every published
endpoint. Also secretless. Deprecated entries are probed but never fail it.

Credentials are never committed — only `<YOUR_API_KEY>` placeholders.
