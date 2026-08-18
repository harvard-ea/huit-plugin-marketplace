# Provisioning: create or adopt a marketplace

For a team standing up its own marketplace. A team does **not** need permission
from a central group to do this — if they can create a repo in their org, they
can run a marketplace.

Read `SKILL.md` §3 first.

## 1. Confirm the team exists

```bash
gh api orgs/<org>/teams --jq '.[] | "\(.slug)  \(.name)  (\(.privacy))"'
```

If the team is not in the list, **do not tell the user it doesn't exist.** A
team with `privacy: secret` is invisible to non-members, so absence from this
list means either "no such team" or "you can't see it". Say exactly that, and
ask them to confirm the slug with a team maintainer.

Once you have a slug:

```bash
gh api orgs/<org>/teams/<slug> --jq '{slug, name, privacy, members_count}'
```

## 2. Confirm membership — read-only, never modify

```bash
gh api "orgs/<org>/teams/<slug>/members" --jq '.[].login'
gh api user --jq .login
```

Match the user's login against the list.

- **Never use `orgs/<org>/teams/<slug>/memberships/<user>`.** It requires the
  `admin:org` scope, which this workflow deliberately does not need. The members
  list gives the same answer with `read:org`.
- **Never `PUT`, `POST`, or `DELETE` against a team endpoint.** This step is a
  check. If the user is not a member, tell them and stop — joining a team is a
  team maintainer's decision, not something to automate around.
- A large team may paginate; add `--paginate` if `members_count` exceeds the
  page you got back.

If the user only wants a marketplace for themselves or an informal group, a
GitHub team is not required at all. Skip to step 3 — repo write permission is
the access control either way.

## 3. Name it

Convention: **`<team>-plugin-marketplace`** (e.g. `iam-plugin-marketplace`).
Predictable names make marketplaces findable across an org.

The *catalog* name — the `name` field in both catalog files, and what users type
after `@` when installing — defaults to the repo name. It can differ, but
keeping them the same avoids a confusing install command.

## 4. Create, or adopt an existing repo

Check first:

```bash
gh api repos/<org>/<name> --jq '{private, permissions}' 2>/dev/null || echo "does not exist"
```

### Creating

**Private is the default.** Create public only if the user explicitly says so
after you've told them what it means: a public marketplace repo is readable by
anyone on the internet, including every endpoint URL and description in it.

```bash
gh repo create <org>/<name> --private \
  --description "<Team> plugin marketplace (Claude Code + Codex)"
```

### Adopting

If the repo already exists, inspect before touching anything:

```bash
gh api repos/<org>/<name>/contents --jq '.[].name'
```

| What you find | What to do |
|---|---|
| Empty, or only `README.md`/`LICENSE` | Seed it (step 5). |
| Both catalogs + `servers/` already | Already a marketplace. Don't re-seed; add missing `schema/` + workflow only. |
| Other content you don't recognize | **Stop and ask.** Describe what's there and let the user decide. Never overwrite a repo you don't understand. |

If it has catalogs but no `schema/` directory, it predates this tooling: add
`schema/` and the workflow, then run the gate and fix whatever it reports.
Report the findings — do not quietly rewrite their entries.

## 5. Seed the repo

Copy from `reference/`:

```
schema/*.json                        <- reference/schemas/*.json  (all of them)
.github/workflows/validate.yml       <- reference/ci/validate.yml
```

Create the two empty catalogs. `<catalog-name>` must be a valid slug and
`<Display Name>` is what users see:

```json
// .claude-plugin/marketplace.json
{ "name": "<catalog-name>", "owner": { "name": "<Display Name>" }, "plugins": [] }
```
```json
// .agents/plugins/marketplace.json
{ "name": "<catalog-name>", "interface": { "displayName": "<Display Name>" }, "plugins": [] }
```

Plus `servers/.gitkeep` and a `README.md` saying what the marketplace is, how to
install from it, and that entries are managed with this skill.

Commit and push:

```bash
git init -q -b main && git add -A && git commit -q -m "seed marketplace"
git remote add origin "https://github.com/<org>/<name>.git" && git push -q -u origin main
```

An empty marketplace is valid — the gate passes with zero entries. Confirm it
went green before handing over.

## 6. Give the team access (ask first)

This is the only step that **writes** anything outside the new repo, so confirm
with the user before running it. It changes who can push to the marketplace; it
does **not** change team membership.

```bash
gh api -X PUT "orgs/<org>/teams/<slug>/repos/<org>/<name>" -f permission=push
```

It requires the user to be a **maintainer** of the team. If it fails with 403 or
404, don't retry or work around it — print this for them to hand to a team
maintainer:

> Please give the `<slug>` team **write** access to `<org>/<name>`:
> repo → Settings → Collaborators and teams → Add team.

Individual collaborators work too, if a team is overkill:

```bash
gh api -X PUT "repos/<org>/<name>/collaborators/<login>" -f permission=push
```

## 7. Verify and hand over

```bash
gh api repos/<org>/<name> --jq '{private, visibility}'          # private unless they chose otherwise
gh api repos/<org>/<name>/actions/secrets --jq '.secrets|length' # MUST be 0
gh run list -R <org>/<name> --limit 1                            # gate green
```

The zero-secrets check matters: this design deliberately needs no PAT and no
credential in any marketplace repo. If something ever adds one, that is a
regression worth flagging, not a convenience.

Then tell the user:

- The repo URL, and that **write access is the access control** — whoever can
  push can publish.
- How to install from it:
  ```
  /plugin marketplace add <org>/<name>
  /plugin install <plugin>@<catalog-name>
  ```
  For a **private** repo, each user needs read access on the repo and working
  git credentials for it; there is nothing extra to configure beyond that.
- That adding entries is `reference/lifecycle.md` — this same skill.

## Branch protection (optional)

For a team that wants review before publishing, protect `main` and require the
`check` status. The lifecycle procedures detect this automatically and switch to
opening PRs. Don't set it up unasked — for a small team it is friction without
benefit, and the gate already runs on pushes to `main`.
