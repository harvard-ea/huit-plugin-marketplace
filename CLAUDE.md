# CLAUDE.md — huit-plugin-marketplace (public data repo)

Guidance for Claude Code (and humans). Read this first when picking up development.

## What this is

A **GitHub-native plugin marketplace** for MCP servers, serving both **Claude
Code** and **OpenAI Codex**. This repo is a **data store** — it holds the
`server.yaml` source files and the generated plugin manifests, both marketplace
catalogs, and a static listing. **There is no build tooling here.**

Generation and PRs are produced by the **submission MCP server**:
`harvard-huit/huit-plugin-marketplace-submit` (validate → generate → open a PR
here). Design + rationale live there in `docs/design.md`, and in the
`mcp-marketplace` repo under `docs/superpowers/specs/` +
`docs/GitHub-Native-Marketplace-Spec.md`.

## Structure

```
servers/<slug>/server.yaml          # SOURCE OF TRUTH (one per MCP server)
servers/<slug>/.claude-plugin/…     # generated (Claude plugin manifest)
servers/<slug>/.codex-plugin/…      # generated (Codex plugin manifest)
servers/<slug>/.mcp.json            # generated (MCP declaration)
.claude-plugin/marketplace.json     # generated catalog (Claude)
.agents/plugins/marketplace.json    # generated catalog (Codex)
docs/index.html                     # generated listing (served by Pages from main)
```

**Only `server.yaml` is hand-authored conceptually** — and even that normally
comes from the submission server. Everything else is **generated**; do not
hand-edit generated files. The submission server regenerates the whole set
(plugin bundle + catalogs + listing) on each submission.

## How content changes

- **Primary path:** a developer uses the submission MCP server, which validates,
  generates, and opens a PR here. A maintainer reviews and merges.
- **Manual PRs** are possible but there is no generator in this repo — a
  hand-authored `server.yaml` must be regenerated (by the submission server or a
  maintainer) before merge. Do not commit a `server.yaml` without its generated
  bundle + updated catalogs.

## Pages

GitHub Pages serves `docs/` from the `main` branch
(Settings → Pages → *Deploy from a branch* → `main` / `docs`). No Actions
workflow. (Enable this in the repo settings once; owner-driven.)

## Install (for reference)

```
/plugin marketplace add harvard-huit/huit-plugin-marketplace      # Claude Code
codex plugin marketplace add harvard-huit/huit-plugin-marketplace # Codex
```
Private repo: installers need GitHub access + an SSO-authorized token for
`harvard-huit`.

## Conventions

- Keep generated files consistent with their `server.yaml` sources — regenerate,
  don't hand-edit. The generator is the `gen/` library in the submission server.
- No secrets here. Auth-requiring servers ship a `<YOUR_API_KEY>` placeholder.

## Status

Exploration. The historical scaffold once held the `hpm` CLI + reusable workflows
+ a contribution skill; those were removed when the tooling moved to the
submission server. This repo is now data-only.
