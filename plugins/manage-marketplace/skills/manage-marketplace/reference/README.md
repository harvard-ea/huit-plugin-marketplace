# Reference assets

Everything the `manage-marketplace` skill needs to produce and check marketplace
files. This directory is **data, not code** — there is no runtime, nothing to
install, and no credential anywhere in it.

| Path | What it is |
|---|---|
| `schemas/` | JSON Schemas (draft-07). The structural contract for every file in a marketplace repo, and what the CI gate validates against. |
| `templates/` | A worked example of each file. Copy and fill; do not invent structure. |
| `ci/validate.yml` | The workflow seeded into a marketplace repo. No secrets, no PAT. |

## Why the schemas are strict

`claude-marketplace.schema.json` sets `additionalProperties: false` on plugin
entries **on purpose**. Claude Code's loader rejects an entry carrying an
unrecognized top-level key, and the endpoint `url` getting copied onto a catalog
entry is the specific mistake that broke installs once. `url` is reserved for
`source` objects of type `"url"`; the endpoint belongs in the entry's
`.mcp.json` and in `server.yaml`, never in the catalog.

Schemas use `pattern`, not `format`, so validation needs no `ajv-formats` plugin
— one less thing to install in a team's CI.

## The two MCP config files

`.mcp.json` is shared by Claude and Codex and carries `type` + `url`. A
credentialed entry additionally gets `.codex-plugin/mcp.json`, which has **no**
`type` and uses `http_headers` where the shared file uses `headers` — Codex
ignores Claude's `headers` key. They have separate schemas because they are
genuinely different shapes; validating one against the other's schema fails.

## Validating

```bash
npx --yes ajv-cli@5 validate --spec=draft7 -s <schema> -d <file.json>
npx --yes js-yaml@4 <file.yaml> > /tmp/d.json   # YAML must be converted first
```

`ajv` exits non-zero both when the *data* is invalid and when the *schema* is
invalid. Those mean very different things, so always read the message — an exit
code alone will report a broken schema as a failing file. `schema-tests/run.sh`
and the CI gate both assert on the message for this reason.

## Self-test

```bash
bash schema-tests/run.sh                                  # templates + rejection fixtures
LIVE=harvard-ea/huit-plugin-marketplace bash schema-tests/run.sh   # + real published files
```

The `LIVE` run is the one that matters: schemas that only accept their own
templates prove nothing. It caught `category`, a key every published
`server.yaml` carries that the retired generator silently dropped.
