# HUIT Plugin Marketplace (public)

A **GitHub-native** plugin marketplace for MCP servers, serving both
**Claude Code** and **OpenAI Codex**. This repo is a **data store**: it holds the
`servers/*/server.yaml` source files and the generated plugin manifests, both
marketplace catalogs, and a static listing. There is no build tooling here —
generation and PRs are produced by the **submission MCP server**
(`harvard-huit/huit-plugin-marketplace-submit`).

> Status: **exploration.** Part of the GitHub-native approach described in
> `docs/GitHub-Native-Marketplace-Spec.md` (in the `mcp-marketplace` repo).

## Use the marketplace

**Claude Code**
```
/plugin marketplace add harvard-huit/huit-plugin-marketplace
/plugin install <slug>@huit-plugin-marketplace
```

**Codex**
```
codex plugin marketplace add harvard-huit/huit-plugin-marketplace
```

Browse the listing on the GitHub Pages site (once enabled), or under `servers/`.

> This repo is **private** during exploration — installing requires GitHub access,
> and your token/SSH key must be **SSO-authorized** to `harvard-huit`.

## Add a server

Submissions go through the **submission MCP server**, not by editing this repo
directly. From your MCP client, ask to *add an MCP server to the marketplace* —
the server validates it, generates the plugin, and opens a PR here for a
maintainer to review. See `harvard-huit/huit-plugin-marketplace-submit`.

## What's in here

```
servers/<slug>/server.yaml          # source of truth (one per MCP server)
servers/<slug>/.claude-plugin/…     # generated plugin manifest (Claude)
servers/<slug>/.codex-plugin/…      # generated plugin manifest (Codex)
servers/<slug>/.mcp.json            # generated MCP declaration
.claude-plugin/marketplace.json     # generated catalog (Claude)
.agents/plugins/marketplace.json    # generated catalog (Codex)
docs/index.html                     # generated listing (served by Pages from main)
```

All files except `server.yaml` are **generated** — they are produced (and kept in
sync) by the submission server. Do not hand-edit generated files.

## Pages

GitHub Pages serves `docs/` directly from the `main` branch
(Settings → Pages → *Deploy from a branch* → `main` / `docs`). No Actions
workflow is required.
