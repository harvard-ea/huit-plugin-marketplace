# Example GitHub

Example MCP server requiring a personal access token. The marketplace never distributes credentials — installers replace the placeholder with their own.

## Install

**Claude Code**
```
/plugin marketplace add harvard-huit/huit-plugin-marketplace
/plugin install example-github@huit-plugin-marketplace
```

**Codex**
```
codex plugin marketplace add harvard-huit/huit-plugin-marketplace
```

## Server

- **Endpoint:** `https://github.example.com/mcp`
- **Transport:** `http`
- **Auth:** `bearer`
- **Tags:** example, developer-tools
- **Homepage:** https://github.com/harvard-huit/huit-plugin-marketplace

## Authentication

This server requires a credential. After installing, replace `<YOUR_API_KEY>` in `.mcp.json` with your own key — credentials are never distributed through the marketplace.

---

_Generated from `server.yaml`. Edit that file and open a PR to change this plugin — do not hand-edit generated files._
