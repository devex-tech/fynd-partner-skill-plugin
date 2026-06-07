# fynd-partner-skill-plugin

> Claude Code skills for Fynd Partner developers — Platform API, Extensions, AI PIM, Konnect, and Themes.

Install once. Ask Claude anything about building on Fynd Commerce.

---

## What's Included

| Skill | Invocation | What It Covers |
|-------|-----------|---------------|
| **fynd-partner** | `/fynd-partner` | Top-level router — routes to the right sub-skill |
| **fynd-platform-api** | `/fynd-platform-api` | 843 REST ops — endpoint discovery, debug, auth, SDK |
| **fynd-extensions** | `/fynd-extensions` | General / Payment / Logistics extension development |
| **fynd-ai-pim** | `/fynd-ai-pim` | SKU enrichment, taxonomy, connectors, rule engine |
| **fynd-konnect** | `/fynd-konnect` | Multi-channel selling, ERP/WMS, 31 APIs |
| **fynd-themes** | `/fynd-themes` | React themes, FDK-CLI, SSR, sections, submission |

---

## Installation

### Option 1 — From this marketplace (recommended)

```bash
/plugin marketplace add devex-tech/fynd-partner-skill-plugin
/plugin install fynd-partner@fynd-partner
```

### Option 2 — Direct from GitHub

```bash
/plugin install devex-tech/fynd-partner-skill-plugin
```

After installation, skills are available as:
```
/fynd-partner
/fynd-platform-api
/fynd-extensions
/fynd-ai-pim
/fynd-konnect
/fynd-themes
```

---

## Usage Examples

```
/fynd-partner which API lists all products in a store?
/fynd-platform-api getting 401 on catalog API
/fynd-extensions how do I build a payment extension?
/fynd-ai-pim how do I set up an inbound connector?
/fynd-konnect how do I connect Amazon to Konnect?
/fynd-themes how do I use ServerFetch for SSR?
```

Or just ask naturally — Claude auto-invokes the right skill:
```
How do I implement HMAC checksum in a Fynd payment extension?
Which Fynd Platform API updates inventory?
How do I sync a theme between two environments?
```

---

## Pair with the MCP Server (Recommended)

For deep reference lookups, also install the `fynd-partner-mcp` MCP server.
The skills use the MCP for detailed docs (64 reference files, 843+ API operations):

**Claude Desktop** (`~/Library/Application Support/Claude/claude_desktop_config.json`):
```json
{
  "mcpServers": {
    "fynd-partner": {
      "command": "npx",
      "args": ["-y", "fynd-partner-mcp"]
    }
  }
}
```

**Cursor** (`~/.cursor/mcp.json`):
```json
{
  "mcpServers": {
    "fynd-partner": {
      "command": "npx",
      "args": ["-y", "fynd-partner-mcp"]
    }
  }
}
```

---

## Repository

[github.com/devex-tech/fynd-partner-skill-plugin](https://github.com/devex-tech/fynd-partner-skill-plugin)
