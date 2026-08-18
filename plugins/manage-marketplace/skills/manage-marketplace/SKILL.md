---
name: manage-marketplace
description: Run a team's own Claude Code + Codex plugin marketplace on GitHub — add, update, deprecate, or remove plugins in it, and list what it holds. Each team owns its own marketplace repo; there is no central marketplace team to go through. Use when someone wants to publish an MCP server or plugin for their team, change or remove a published entry, or see what a marketplace contains. Triggers: "add this server to our marketplace", "publish this MCP server", "update our marketplace entry", "deprecate/remove <plugin>", "what's in our marketplace".
---

# Manage a plugin marketplace

A marketplace is an ordinary GitHub repo holding two catalogs and one directory
per plugin. Both Claude Code and Codex install from the same repo.

```
.claude-plugin/marketplace.json      # Claude catalog
.agents/plugins/marketplace.json     # Codex catalog
servers/<slug>/server.yaml           # source of truth for one entry
servers/<slug>/…                     # manifests derived from server.yaml
schema/                              # the schemas CI validates against
```

**Access control is repo write permission.** Whoever can push can publish. There
is no allowlist, no central registry, and no approval queue.

## 1. Identify the target repo

In this order — never guess, never fall back to a default:

1. An `owner/repo` the user gave you.
2. If the working directory is inside a marketplace repo (it has
   `.claude-plugin/marketplace.json`), use `git remote get-url origin`.
3. Otherwise **ask**. If they don't know it, `gh repo list <org> --limit 100
   --json name,description | grep -i marketplace` helps them find it.

Confirm you can write to it before doing any work:

```bash
gh api repos/<owner>/<repo> --jq '{private,permissions}'
```

If `permissions.push` is false, stop and tell the user — they need write access
from whoever administers that repo. Do not attempt a fork-and-PR workaround.

## 2. Choose the procedure

| The user wants to | Read |
|---|---|
| add, update, deprecate, remove, or list entries | `reference/lifecycle.md` |
| create a new marketplace, or adopt an existing repo as one | `reference/provisioning.md` |

Read the reference file before acting. Do not work from memory of this page.

## 3. Rules that always apply

- **Never put `url` on a catalog plugin entry.** Claude Code's loader rejects
  entries with unrecognized keys, and this specific mistake broke installs once.
  The endpoint lives in `server.yaml` and the entry's `.mcp.json`.
- **Update both catalogs in the same commit.** A repo serves Claude and Codex
  together; changing one and not the other leaves it half-broken.
- **Slugs** are lowercase alphanumeric + hyphens, no leading/trailing hyphen,
  and must not be one of: `claude`, `anthropic`, `codex`, `openai`, `plugin`,
  `plugins`, `marketplace`, `mcp`, `agent`, `agents`, `skill`, `skills`,
  `help`, `init`, `config`, `servers`.
- **Endpoints and homepages must be `https://`.** No `http://`, no whitespace.
- **Never commit a real credential.** Credentialed servers get a
  `<YOUR_API_KEY>` placeholder the installing user replaces themselves.
- **Validate before every commit** (§4). Do not push unvalidated files.
- **`server.yaml` is the source of truth.** Change it first, then bring the
  derived files into line with it — not the other way round.

## 4. Validate before committing

From the root of the marketplace repo:

```bash
npx --yes ajv-cli@5 validate --spec=draft7 \
  -s schema/claude-marketplace.schema.json -d .claude-plugin/marketplace.json
npx --yes ajv-cli@5 validate --spec=draft7 \
  -s schema/codex-marketplace.schema.json -d .agents/plugins/marketplace.json
# server.yaml is YAML — convert first:
npx --yes js-yaml@4 servers/<slug>/server.yaml > /tmp/s.json
npx --yes ajv-cli@5 validate --spec=draft7 -s schema/server.schema.json -d /tmp/s.json
```

`ajv` exits non-zero both when the **data** is invalid and when the **schema**
itself is invalid. **Read the message, not the exit code.** `schema … is
invalid` means the schema is broken, not the file — that is a different problem
and must not be reported as a bad entry.

If the repo has no `schema/` directory it predates this tooling; see
`reference/provisioning.md` on adopting it.
