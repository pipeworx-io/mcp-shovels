# mcp-shovels

Shovels MCP — wraps the Shovels.ai API (shovels.ai)

Part of [Pipeworx](https://pipeworx.io) — an MCP gateway connecting AI agents to 1857+ live data sources. This is an independent, unofficial integration — not affiliated with, endorsed by, or published by the upstream provider.

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

Both URLs reach the same gateway and the same 1857+ data sources. The
only difference is which pack's tools are listed **directly**; `ask_pipeworx`
reaches all of them from either one.

## No MCP client? Call it over HTTP

This pack takes your own API key (`_apiKey`) — we don't front one for it, so there's no curl here that would run without it. Inspect any tool: `GET https://gateway.pipeworx.io/v1/tools/shovels_address_search`. Find one: `POST https://gateway.pipeworx.io/v1/tools/search_packs` with `{"query":"..."}`.

## Run it from npm

`@pipeworx/mcp-shovels` is a local stdio bridge to the Pipeworx gateway: your MCP client
starts it with `npx`, and it forwards every call to the same
`https://gateway.pipeworx.io/shovels/mcp` endpoint shown above. It needs network access;
there is no offline mode.

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

Without a key it calls anonymously, capped at 50 calls a day. A free account
raises that to 200 a day: get a key at https://pipeworx.io/signup and add
`"env": { "PIPEWORX_API_KEY": "<your key>" }` to the server entry.
The key is sent as a Bearer token, the same way a hosted connection sends it,
so calls are metered and billed exactly as they are there.

It lists **only** this pack's tools by default. Set `PIPEWORX_META_TOOLS=1` to
also list `ask_pipeworx` and the other gateway-wide tools.

Behind an HTTPS proxy: Node's built-in `fetch` does not read `HTTPS_PROXY` by
default. Newer Node releases honour it when `NODE_USE_ENV_PROXY=1` is set; check
your Node version's documentation.

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
