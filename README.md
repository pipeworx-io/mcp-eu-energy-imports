# @pipeworx/eu-energy-imports

Where Germany and each EU country import gas, oil and coal from, how much of it is Russian, how import-dependent they are, and what their energy mix looks like.

Part of [Pipeworx](https://pipeworx.io) — an MCP gateway connecting AI agents to 1879+ live data sources. This is an independent, unofficial integration — not affiliated with, endorsed by, or published by the upstream provider.

## Tools

- `import_origins(fuel, geo?, period?, top_n?, product?)` — ranked suppliers of natural gas, crude oil or coal to a country or the EU, with each supplier's share and Russia's share. Annual by default (latest published year); gas and oil also take a month.
- `russia_import_trend(fuel, geo?, from_year?, to_year?)` — the Russian share of imports, year by year, for gas, oil or coal.
- `import_dependency_rate(energy?, geo?, from_year?, to_year?)` — percentage of energy needs met by net imports (total, gas, oil or coal), by year.
- `energy_mix(geo?, year?)` — gross available energy by fuel (gas, oil, coal, renewables, nuclear, waste, net electricity and heat) in terajoules and percent.

`geo` is a 2-letter country code (`DE`, `FR`, `IT`; `EL` for Greece) or `EU` for the EU-27 total.

## Auth

Keyless.

## Data sources

- Eurostat, energy statistics — imports of natural gas, oil and solid fossil fuels by partner country; energy import dependency; complete energy balances. Every response carries `data_as_of` (the publisher's last update) and the latest period is provisional and revised later.

Called live per request; nothing is stored. Imports are reported by the importing country by partner (origin) country. Gas is in terajoules (gross calorific value) annually and million cubic metres monthly; oil and coal in thousand tonnes. Germany's own national statistics (AGEB, BMWK) are not included.

## Quick Start

Add to your MCP client (Claude Desktop, Cursor, Windsurf, etc.):

```json
{
  "mcpServers": {
    "eu-energy-imports": {
      "url": "https://gateway.pipeworx.io/eu-energy-imports/mcp"
    }
  }
}
```

### What this endpoint actually serves

`tools/list` at `https://gateway.pipeworx.io/eu-energy-imports/mcp` returns the tools in the table
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

Both URLs reach the same gateway and the same 1879+ data sources. The
only difference is which pack's tools are listed **directly**; `ask_pipeworx`
reaches all of them from either one.

## No MCP client? Call it over HTTP

```bash
curl -X POST https://gateway.pipeworx.io/v1/tools/import_origins \
  -H 'Content-Type: application/json' \
  -d '{"fuel":"gas","geo":"DE","period":"2024","top_n":5}'
```

No account needed for the first calls. Inspect any tool: `GET https://gateway.pipeworx.io/v1/tools/import_origins`. Find one: `POST https://gateway.pipeworx.io/v1/tools/search_packs` with `{"query":"..."}`.

## Standalone (no gateway account)

This package also runs as a local stdio MCP server — no Pipeworx account, no
gateway round-trip:

```json
{
  "mcpServers": {
    "eu-energy-imports": {
      "command": "npx",
      "args": ["-y", "@pipeworx/mcp-eu-energy-imports"]
    }
  }
}
```

Or run it directly to confirm it starts:

```bash
npx -y @pipeworx/mcp-eu-energy-imports
```

It speaks MCP over stdin/stdout and answers `initialize`/`tools/list`/`tools/call`
for **only** this pack's tools — none of the shared meta-tools the gateway
connection above adds. Same source, same tools, no ask_pipeworx routing.

## Using with ask_pipeworx

Instead of calling tools directly, you can ask questions in plain English —
this works on the pack endpoint above as well as on the full gateway:

```
ask_pipeworx({ question: "your question about Eu Energy Imports data" })
```

The gateway picks the right tool and fills the arguments automatically.

## More

- [Docs and guides](https://pipeworx.io/docs)
- [pipeworx.io](https://pipeworx.io)

## License

MIT
