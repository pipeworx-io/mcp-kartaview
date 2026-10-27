# @pipeworx/kartaview

KartaView (formerly OpenStreetCam) — open, crowdsourced street-level imagery
contributed by OpenStreetMap mappers, CC BY-SA, with GPS position, compass
heading and capture date on every frame.

Part of [Pipeworx](https://pipeworx.io) — an MCP gateway connecting AI agents to 1882+ live data sources. This is an independent, unofficial integration — not affiliated with, endorsed by, or published by the upstream provider.

## Tools

- `kartaview_nearby_sequences(lat, lng, radius_m)` — street-level photos near a point, grouped into the drives that captured them, nearest first.
- `kartaview_sequence_photos(sequence_id)` — every frame of one drive in capture order, with JPEG URLs.

## Auth

Keyless.

## Data sources

- <https://api.openstreetcam.org/1.0/list/nearby-photos/> — POST form `lat`, `lng`, `radius`.
- <https://api.openstreetcam.org/1.0/sequence/photo-list/> — POST form `sequenceId`.

Notes the next person would otherwise rediscover:

- **The host serves two APIs and only the 1.0 one is anonymous.** The 2.0 read
  endpoints (`/2.0/sequence/?lat=&lng=`) answer `{"status":{"apiCode":401,
  "apiMessage":"You have to be authenticated..."}}` — inside an HTTP **200**.
  So a 200 from this host proves nothing; `status.apiCode` must be `600`.
  `kvPost()` checks it on every call.
- Both endpoints are **POST with a form body**. The GET forms 400.
- Photo fields are storage-relative paths (`storage13/files/photo/...`), not
  URLs, and the storage number in the path is not the hostname that serves it.
  Prefixing `https://api.openstreetcam.org/` redirects to the right storage
  host; building `https://storage13.openstreetcam.org/storage13/...` 404s.
- `nearby-photos` returns PHOTOS, not sequences, and has no sequence endpoint
  that is keyless — so the pack groups by `sequence_id` itself and computes the
  distance to each hit, which the upstream does not return.
- Coverage is volunteer-contributed: dense in cities with active OSM
  communities, empty elsewhere. An empty result is a coverage fact, not an
  error, and the tool says so.

## Quick Start

Add to your MCP client (Claude Desktop, Cursor, Windsurf, etc.):

```json
{
  "mcpServers": {
    "kartaview": {
      "url": "https://gateway.pipeworx.io/kartaview/mcp"
    }
  }
}
```

### What this endpoint actually serves

`tools/list` at `https://gateway.pipeworx.io/kartaview/mcp` returns the tools in the table
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

Both URLs reach the same gateway and the same 1882+ data sources. The
only difference is which pack's tools are listed **directly**; `ask_pipeworx`
reaches all of them from either one.

## No MCP client? Call it over HTTP

```bash
curl -X POST https://gateway.pipeworx.io/v1/tools/kartaview_nearby_sequences \
  -H 'Content-Type: application/json' \
  -d '{"lat":52.3676,"lng":4.9041,"radius_m":300,"limit":3}'
```

No account needed for the first calls. Inspect any tool: `GET https://gateway.pipeworx.io/v1/tools/kartaview_nearby_sequences`. Find one: `POST https://gateway.pipeworx.io/v1/tools/search_packs` with `{"query":"..."}`.

## Standalone (no gateway account)

This package also runs as a local stdio MCP server — no Pipeworx account, no
gateway round-trip:

```json
{
  "mcpServers": {
    "kartaview": {
      "command": "npx",
      "args": ["-y", "@pipeworx/mcp-kartaview"]
    }
  }
}
```

Or run it directly to confirm it starts:

```bash
npx -y @pipeworx/mcp-kartaview
```

It speaks MCP over stdin/stdout and answers `initialize`/`tools/list`/`tools/call`
for **only** this pack's tools — none of the shared meta-tools the gateway
connection above adds. Same source, same tools, no ask_pipeworx routing.

## Using with ask_pipeworx

Instead of calling tools directly, you can ask questions in plain English —
this works on the pack endpoint above as well as on the full gateway:

```
ask_pipeworx({ question: "your question about Kartaview data" })
```

The gateway picks the right tool and fills the arguments automatically.

## More

- [Docs and guides](https://pipeworx.io/docs)
- [pipeworx.io](https://pipeworx.io)

## License

MIT
