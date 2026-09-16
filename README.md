# mcp-shovels

Shovels MCP — wraps the Shovels.ai API (shovels.ai)

Part of [Pipeworx](https://pipeworx.io) — an MCP gateway connecting AI agents to 1576+ live data sources.

## Tools

| Tool | Description |
|------|-------------|
| `shovels_address_search` | Resolve a US address to Shovels records, each carrying a `geo_id` — the geolocation ID you pass to shovels_permits_search / shovels_contractors_search to scope those queries. Results are ordered by relevance in USPS notation. Example: shovels_address_search({ q: "1600 Pennsylvania Ave NW, Washington DC", _apiKey: "your-key" }) |
| `shovels_permits_search` | Search US building permits scoped to a location. Requires a `geo_id` (a US state code like "CA", a ZIP code, or a Shovels geo_id from shovels_address_search) plus a permit date range (permit_from / permit_to, YYYY-MM-DD). Monetary values (job_value, fees, market value) are returned in both cents and dollars. Example: shovels_permits_search({ geo_id: "CA", permit_from: "2024-01-01", permit_to: "2024-12-31", property_type: "residential", _apiKey: "your-key" }) |
| `shovels_contractors_search` | Search US construction contractors scoped to a location, with license, work history and aggregate job value. Requires a `geo_id` (state code, ZIP, or Shovels geo_id from shovels_address_search) plus a permit date range (permit_from / permit_to, YYYY-MM-DD). Job-value fields are returned in both cents and dollars. Example: shovels_contractors_search({ geo_id: "94103", permit_from: "2023-01-01", permit_to: "2024-12-31", contractor_name: "solar", _apiKey: "your-key" }) |

## Quick Start

Add to your MCP client (Claude Desktop, Cursor, Windsurf, etc.):

```json
{
  "mcpServers": {
    "shovels": {
      "url": "https://gateway.pipeworx.io/shovels/mcp"
    }
  }
}
```

### What this endpoint actually serves

`tools/list` at `https://gateway.pipeworx.io/shovels/mcp` returns the tools in the table
above **plus the shared Pipeworx meta-tools** — `ask_pipeworx`,
`discover_tools`, `search_within`, `remember`/`recall` and the rest of the
gateway-wide set. So the tool count you see is larger than this table: a
single-pack endpoint currently lists roughly 30 shared tools alongside the
pack's own. The connection's `initialize` response states its exact scope, and
is the authoritative answer for a given day.

This is deliberate, not multiplexing by accident. The meta-tools are what let a
scoped connection answer a question this pack does not cover — via
`ask_pipeworx`, which routes across the whole catalog — without you adding a
second MCP server. There is currently no way to mount a pack endpoint without
them; if the extra schemas cost you more context than the routing is worth,
connect to the full gateway once rather than to several pack endpoints.

Or connect to the full Pipeworx gateway to get every pack's tools listed
directly, instead of just this one's:

```json
{
  "mcpServers": {
    "pipeworx": {
      "url": "https://gateway.pipeworx.io/mcp"
    }
  }
}
```

Both URLs reach the same gateway and the same 1576+ data sources. The
only difference is which pack's tools are listed **directly**; `ask_pipeworx`
reaches all of them from either one.

## Standalone (no gateway account)

This package also runs as a local stdio MCP server — no Pipeworx account, no
gateway round-trip:

```json
{
  "mcpServers": {
    "shovels": {
      "command": "npx",
      "args": ["-y", "@pipeworx/mcp-shovels"]
    }
  }
}
```

Or run it directly to confirm it starts:

```bash
npx -y @pipeworx/mcp-shovels
```

It speaks MCP over stdin/stdout and answers `initialize`/`tools/list`/`tools/call`
for **only** this pack's tools — none of the shared meta-tools the gateway
connection above adds. Same source, same tools, no ask_pipeworx routing.

## Using with ask_pipeworx

Instead of calling tools directly, you can ask questions in plain English —
this works on the pack endpoint above as well as on the full gateway:

```
ask_pipeworx({ question: "your question about Shovels data" })
```

The gateway picks the right tool and fills the arguments automatically.

## More

- [Docs and guides](https://pipeworx.io/docs)
- [pipeworx.io](https://pipeworx.io)

## License

MIT
