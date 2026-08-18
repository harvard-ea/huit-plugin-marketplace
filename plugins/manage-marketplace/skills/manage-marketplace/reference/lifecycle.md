# Lifecycle: add, update, deprecate, remove, list

Procedures for changing what a marketplace publishes. Read `SKILL.md` §3 first —
those rules apply to everything here.

## Working loop

Every change follows the same shape:

```bash
# 1. Clone (shallow is fine; work in a temp dir, never in the user's project)
cd "$(mktemp -d)" && gh repo clone <owner>/<repo> . -- --depth=1 && cd .

# 2. Make the change (procedures below)
# 3. Validate — SKILL.md §4
# 4. Commit and push
git add -A && git commit -m "<message>"
git push origin main
```

**If the push is rejected by branch protection**, don't fight it — that repo
wants review. Switch to a branch and open a PR:

```bash
git checkout -b <verb>/<slug>
git push -u origin <verb>/<slug>
gh pr create --fill --title "<verb>: <name>" --body "<what changed and why>"
```

Report the PR URL. Never merge it yourself unless the user says they are the
reviewer and asks you to.

Prefer pushing to `main` when it is allowed: on a repo the user administers, a
PR to themselves is ceremony, and the CI gate runs on pushes to `main` anyway.

## Add an entry

### Collect

Required: `name`, `description`, and — for an MCP server — the `https://`
endpoint `url`. Strongly encourage `tags` and an `https://` `homepage`; they are
what make an entry findable. Ask for `category` if the marketplace already uses
categories (check the existing catalog).

If the user points at a repo or README instead of listing fields, read it,
extract the fields, and **confirm what you extracted** before writing anything.

Derive the slug from the name: lowercase, non-alphanumerics to hyphens, trimmed.
Check it against the reserved list in `SKILL.md` §3, and check it is not already
taken:

```bash
ls servers/            # a directory named <slug> means this is an UPDATE, not an add
```

If the slug exists, stop and confirm with the user that they mean to update it.
Never silently overwrite an existing entry.

### Write the files

Copy the matching file from `reference/templates/` and fill it in. Do not invent
structure.

For **`kind: mcp-server`** (the default):

| File | From template | Notes |
|---|---|---|
| `servers/<slug>/server.yaml` | `server.yaml` | Source of truth. Record `owner:` as the user's email. |
| `servers/<slug>/.claude-plugin/plugin.json` | `claude-plugin.json` | `author.name` = the marketplace owner. |
| `servers/<slug>/.codex-plugin/plugin.json` | `codex-plugin.json` | See auth table below for `mcpServers`. |
| `servers/<slug>/.mcp.json` | `mcp.json` | Shared by both ecosystems. |
| `servers/<slug>/.codex-plugin/mcp.json` | `codex-mcp.json` | **Credentialed entries only** (`bearer`/`api_key`). See the auth table. |
| `servers/<slug>/README.md` | — | Written by hand; see below. |

For **`kind: skill`** (a plugin shipping skills/commands, no MCP endpoint):
omit `.mcp.json`, omit `mcpServers` from the Codex manifest, and use
`server-skill.yaml` as the `server.yaml` template. Everything else is the same.

### Auth wiring

This is the part most easily got wrong. Claude reads `headers`; Codex reads
`http_headers`. They are not interchangeable.

| `auth.type` | `.mcp.json` | Extra `.codex-plugin/mcp.json`? | Codex `mcpServers` | Codex `policy.authentication` |
|---|---|---|---|---|
| `none` | `{type, url}` | no | `./.mcp.json` | **omit the field** |
| `oauth` | `{type, url}` | no | `./.mcp.json` | `ON_FIRST_USE` |
| `bearer` | + `headers: {<header>: "Bearer <YOUR_API_KEY>"}` | **yes**, with `http_headers` | `./.codex-plugin/mcp.json` | `ON_INSTALL` |
| `api_key` | + `headers: {<header>: "<YOUR_API_KEY>"}` | **yes**, with `http_headers` | `./.codex-plugin/mcp.json` | `ON_INSTALL` |

- Default header name is `Authorization` unless `auth.header_name` says otherwise.
- Only `bearer` gets the literal `Bearer ` prefix; `api_key` carries the raw token.
- There is **no `NONE`** value for `policy.authentication`. For a no-auth entry
  omit the key entirely — writing `"NONE"` is invalid and CI will reject it.
- The credential is always the placeholder `<YOUR_API_KEY>`, never a real value.

### Add to both catalogs

Append an entry to `.claude-plugin/marketplace.json`:

```json
{
  "name": "<slug>",
  "source": "./servers/<slug>",
  "description": "<description>",
  "version": "<version>",
  "tags": ["…"],
  "category": "<category>",
  "homepage": "https://…"
}
```

Only these keys are permitted: `name`, `source`, `description`, `version`,
`category`, `tags`, `strict`, `relevance`, `author`, `homepage`, `repository`,
`license`, `keywords`. **Anything else — `url` above all — makes Claude Code
reject the entry.**

And to `.agents/plugins/marketplace.json`:

```json
{
  "name": "<slug>",
  "source": "./servers/<slug>",
  "policy": { "installation": "AVAILABLE" }
}
```

Keep both `plugins` arrays sorted by `name` so diffs stay readable.

### The per-plugin README

`servers/<slug>/README.md` is what a human sees browsing the repo. Include the
display name, the description, install commands for both ecosystems, and the
endpoint/transport/auth facts:

````markdown
# <Name>

<description>

## Install

**Claude Code**
```
/plugin marketplace add <owner>/<repo>
/plugin install <slug>@<marketplace-name>
```

**Codex**
```
codex plugin marketplace add <owner>/<repo>
```

## Server

- **Endpoint:** `<url>`
- **Transport:** `http`
- **Auth:** `none`
````

`<marketplace-name>` is the `name` field of `.claude-plugin/marketplace.json`,
not the repo name — they often differ. For a credentialed server, add a section
telling the user to replace `<YOUR_API_KEY>` in `.mcp.json` (Claude) or
`.codex-plugin/mcp.json` (Codex) with their own key.

Commit message: `add: <name>`.

## Update an entry

1. Edit `servers/<slug>/server.yaml` first.
2. Bring every derived file into line with it — including the catalog entries if
   `description`, `version`, `tags`, `category`, or `homepage` changed.
3. Encourage a `version` bump; say so if they leave it unchanged.
4. If the endpoint or auth changed, say so explicitly in the commit message —
   anyone who already installed the plugin is affected.

Commit message: `update: <name>`.

## Deprecate an entry

Deprecating keeps the plugin installed and installable but marks it as on its
way out. Prefer this over removal when the server still works.

1. In `server.yaml`: set `deprecated: true` and `deprecated_reason: "<why>"`.
2. At the top of `servers/<slug>/README.md`, immediately under the heading:
   `> **DEPRECATED.** <reason>`
3. Leave both catalog entries in place — that is the whole point.

To reinstate, remove `deprecated`/`deprecated_reason` and the README banner.

Commit message: `deprecate: <name>` (or `reinstate: <name>`).

## Remove an entry

Removal breaks existing installs. Confirm the user means removal and not
deprecation before doing it.

```bash
git rm -r servers/<slug>
```

Then delete the entry from **both** catalogs. Verify nothing still references
the slug:

```bash
grep -rn "<slug>" . --exclude-dir=.git || echo "clean"
```

Commit message: `remove: <name>`.

## List what a marketplace holds

Read-only; no clone needed.

```bash
gh api repos/<owner>/<repo>/contents/.claude-plugin/marketplace.json \
  --jq '.content' | base64 -d \
  | node -e "const c=JSON.parse(require('fs').readFileSync(0));
      console.log(c.name);
      (c.plugins||[]).forEach(p=>console.log('  '+p.name+'  '+(p.version||'')+'  '+p.description));"
```

To see whether any entry is deprecated — which the catalogs do not record — read
the `server.yaml` files:

```bash
for s in $(gh api repos/<owner>/<repo>/contents/servers --jq '.[].name'); do
  gh api "repos/<owner>/<repo>/contents/servers/$s/server.yaml" --jq '.content' \
    | base64 -d | grep -q '^deprecated: true' && echo "$s [deprecated]" || echo "$s"
done
```

## After any change

Confirm CI went green, and report it honestly if it did not:

```bash
gh run list -R <owner>/<repo> --limit 1
gh run view -R <owner>/<repo> --log-failed   # only if it failed
```
