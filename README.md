# HUIT Plugin Marketplace

A GitHub-native plugin marketplace serving both **Claude Code** and **OpenAI
Codex** from one repo. It holds MCP servers and skill plugins.

## Install from this marketplace

**Claude Code**
```
/plugin marketplace add harvard-ea/huit-plugin-marketplace
/plugin install <name>@huit-plugin-marketplace
```

**Codex**
```
codex plugin marketplace add harvard-ea/huit-plugin-marketplace
```

Browse what's here under `servers/` and `plugins/`, or read
`.claude-plugin/marketplace.json`.

## Run your own marketplace

**Your team does not need to publish here.** Any team that can create a repo in
its GitHub org can run its own marketplace — there is no central marketplace
team to go through, and no approval queue.

Install the toolkit from this marketplace and it will walk you through it:

```
/plugin marketplace add harvard-ea/huit-plugin-marketplace
/plugin install manage-marketplace@huit-plugin-marketplace
```

Then ask Claude to set up a marketplace for your team. It confirms your GitHub
team (read-only — it never changes membership), creates a **private** repo by
default, seeds it, and manages entries from then on.

## Layout

```
.claude-plugin/marketplace.json      # Claude catalog
.agents/plugins/marketplace.json     # Codex catalog
servers/<name>/                      # an entry providing an MCP server
plugins/<name>/                       # an entry providing skills only
schema/                              # JSON Schemas the CI gate validates against
```

Each entry's `server.yaml` is the source of truth; the manifests beside it are
derived from it.

## Changing what's published

Use the `manage-marketplace` skill (above) — it is the same tool whether you're
publishing here or running your own marketplace. Write access to this repo is
what allows publishing; there is no allowlist.

CI validates every change against the schemas in `schema/`. It needs **no
repository secret** — see `CONTRIBUTING.md`.

Credentials are never stored here. An entry needing a credential ships a
`<YOUR_API_KEY>` placeholder that each user replaces locally.
