# @pipeworx/reliefweb

UN OCHA's humanitarian information service — situation reports, needs
assessments, appeals and analyses from ~4,000 humanitarian organisations, plus
the curated disaster register with GLIDE numbers.

Part of [Pipeworx](https://pipeworx.io) — an MCP gateway connecting AI agents to 1679+ live data sources.

## Tools

- `reliefweb_reports(query?, country?, disaster_type?, source?, theme?, format?, from_date?, to_date?, include_body?, limit?, offset?)`
- `reliefweb_disasters(query?, country?, status?, disaster_type?, include_description?, limit?, offset?)`
- `reliefweb_report(report_id, max_body_chars?)` — one report in full.

## Auth

**BYO only**, and the credential is not shaped like a key.

The API is free and charges nothing. What it requires, since **1 November 2025**,
is a **pre-approved `appname`** query parameter on every call. An arbitrary
string is refused with `403 AccessDeniedHttpException`. Verified live
2026-09-17: `appname=pipeworx`, `appname=reliefweb`, `appname=test` all 403 on
v2; v1 additionally answers `410` (decommissioned).

Request one at <https://apidoc.reliefweb.int/parameters#appname>. ReliefWeb
reviews the request and emails the approved string, which must combine your
organisation name, your purpose and random characters. Pass it as `_apiKey`.

Pipeworx holds no approved appname. When one arrives it is
`PLATFORM_RELIEFWEB_APPNAME`; add `platformKeyEnv` to the pack manifest in the
**same change** that lands the value, never before — declaring it first routes
humanitarian questions to a tool that can only refuse.

ReliefWeb rate-limits **by appname**, so a shared one is a shared bucket.

## Data sources

- `POST https://api.reliefweb.int/v2/reports?appname=…`
- `POST https://api.reliefweb.int/v2/disasters?appname=…`
- Field tables: <https://apidoc.reliefweb.int/fields-tables>

Things worth knowing:

- **v1 is gone.** It answers `410` telling you to use v2. Anything written
  against `api.reliefweb.int/v1` is dead.
- The public site's RSS feeds are not a workaround: `reliefweb.int/updates/rss.xml`
  answers `202` with a zero-byte body from server egress, browser UA or not.
- Filters take one `{field, value}` or an `{operator, conditions[]}` group —
  a bare array of filters is rejected.
- Date filters go on `date.original` (publication) as `{from, to}` with full
  ISO timestamps, not bare dates.
- Taxonomy fields (`source`, `country`, `theme`, `format`, `disaster_type`) come
  back as arrays of objects; the pack flattens them to names.
- Report bodies are long. `reliefweb_reports` omits them unless you ask;
  `reliefweb_report` is the one that returns full text.

## Verified

2026-09-17 — refusal paths only, by design. Both the no-key refusal and the
upstream 403-on-unapproved-appname path were exercised live and both say
"requires an API key". Query paths cannot be verified until an appname is approved.

## Quick Start

Add to your MCP client (Claude Desktop, Cursor, Windsurf, etc.):

```json
{
  "mcpServers": {
    "reliefweb": {
      "url": "https://gateway.pipeworx.io/reliefweb/mcp"
    }
  }
}
```

### What this endpoint actually serves

`tools/list` at `https://gateway.pipeworx.io/reliefweb/mcp` returns the tools in the table
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

Both URLs reach the same gateway and the same 1679+ data sources. The
only difference is which pack's tools are listed **directly**; `ask_pipeworx`
reaches all of them from either one.

## No MCP client? Call it over HTTP

```bash
curl -X POST https://gateway.pipeworx.io/v1/tools/reliefweb_reports \
  -H 'Content-Type: application/json' \
  -d '{"country":"Sudan","format":"Situation Report","limit":10}'
```

No account needed for the first calls. Inspect any tool: `GET https://gateway.pipeworx.io/v1/tools/reliefweb_reports`. Find one: `POST https://gateway.pipeworx.io/v1/tools/search_packs` with `{"query":"..."}`.

## Standalone (no gateway account)

This package also runs as a local stdio MCP server — no Pipeworx account, no
gateway round-trip:

```json
{
  "mcpServers": {
    "reliefweb": {
      "command": "npx",
      "args": ["-y", "@pipeworx/mcp-reliefweb"]
    }
  }
}
```

Or run it directly to confirm it starts:

```bash
npx -y @pipeworx/mcp-reliefweb
```

It speaks MCP over stdin/stdout and answers `initialize`/`tools/list`/`tools/call`
for **only** this pack's tools — none of the shared meta-tools the gateway
connection above adds. Same source, same tools, no ask_pipeworx routing.

## Using with ask_pipeworx

Instead of calling tools directly, you can ask questions in plain English —
this works on the pack endpoint above as well as on the full gateway:

```
ask_pipeworx({ question: "your question about Reliefweb data" })
```

The gateway picks the right tool and fills the arguments automatically.

## More

- [Docs and guides](https://pipeworx.io/docs)
- [pipeworx.io](https://pipeworx.io)

## License

MIT
