# @pipeworx/apm-terminals

Container and vessel data direct from the terminal operator APM Terminals —
import container status/holds, export booking status, empty-return status,
vessel arrival/departure schedules, and full container event history, for
any container/booking/vessel at one of APM Terminals' facilities.

Part of [Pipeworx](https://pipeworx.io) — an MCP gateway connecting AI agents to 1867+ live data sources. This is an independent, unofficial integration — not affiliated with, endorsed by, or published by the upstream provider.

## Tools

- `import_availability(asset_id, facility_code)` — container status, holds,
  discharge/yard/appointment data for one or more container or Bill of Lading
  numbers.
- `export_booking_enquiry(asset_id, facility_code)` — booking receiving
  window, cargo cut-off, vessel ETD and per-container load readiness for one
  or more booking reference numbers.
- `empty_container_returns(asset_id, facility_code)` — whether container(s)
  have been returned empty and the gate-in time.
- `terminal_vessel_schedule(asset_id, facility_code, inbound_voyage_number?,
  outbound_voyage_number?)` — vessel ETA/ETD, cut-offs, berth and operator
  for one vessel (by name or Lloyds code) at one terminal.
- `container_event_history(asset_id, facility_code)` — full timestamped
  lifecycle events for one or more containers.

## Auth

Platform key (`PLATFORM_APMTERMINALS_KEY`) with BYO override via `_apiKey`.
Format: `<consumer_key>:<consumer_secret>` from an OAuth2 Client Credentials
Grant app.

**Registration (not yet done — no platform key is configured):**
1. Sign up for a free account at the vendor's developer portal
   (`developer.apmterminals.com`).
2. Log in, open "Plans", and accept the free Data Services Plan — no charge,
   no obligation, cancel anytime.
3. Open "Apps" → "Add App" to register an app and receive a Consumer
   Key/Secret pair.
4. Each API carries its own free 90-day trial from the date the plan is
   accepted; production calls after that draw down a purchased credit
   bundle (see the vendor's Pricing page) — there is no automatic renewal
   or card charge during the trial.
5. Production calls require an OAuth2 bearer token exchanged from the
   Consumer Key/Secret (this pack does the exchange and caches the token).

The vendor's sandbox (`api-sandbox.apmterminals.com`) is **keyless** and was
used to verify every tool's request/response shape live — it returns fixed
sample data (dated 2019–2021), not real shipments, so it is not wired into
the shipped tools; production needs the key above.

One API this vendor publishes, Truck Appointment scheduling (TERMPoint), is
deliberately NOT implemented here: the vendor's own docs require every
end-user to be individually registered and onboarded on TERMPoint by a
specific trucking company, so it has no general "look anything up" shape —
out of place in a shared pack.

## Data sources

- APM Terminals, container and vessel terminal-operating-system data —
  live operational data for containers, bookings and vessel calls at the
  terminals APM Terminals operates (~60+ facilities worldwide, coverage
  rolled out gradually per terminal).

This is an independent, direct build against the terminal operator's own
published API — not a successor to, re-skin of, or replacement for any
other pack.

## Quick Start

Add to your MCP client (Claude Desktop, Cursor, Windsurf, etc.):

```json
{
  "mcpServers": {
    "apm-terminals": {
      "url": "https://gateway.pipeworx.io/apm-terminals/mcp"
    }
  }
}
```

### What this endpoint actually serves

`tools/list` at `https://gateway.pipeworx.io/apm-terminals/mcp` returns the tools in the table
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

Both URLs reach the same gateway and the same 1867+ data sources. The
only difference is which pack's tools are listed **directly**; `ask_pipeworx`
reaches all of them from either one.

## No MCP client? Call it over HTTP

```bash
curl -X POST https://gateway.pipeworx.io/v1/tools/import_availability \
  -H 'Content-Type: application/json' \
  -d '{"asset_id":"WFHU5080179,EGHU9044403","facility_code":"USLAX"}'
```

No account needed for the first calls. Inspect any tool: `GET https://gateway.pipeworx.io/v1/tools/import_availability`. Find one: `POST https://gateway.pipeworx.io/v1/tools/search_packs` with `{"query":"..."}`.

## Run it from npm

`@pipeworx/mcp-apm-terminals` is a local stdio bridge to the Pipeworx gateway: your MCP client
starts it with `npx`, and it forwards every call to the same
`https://gateway.pipeworx.io/apm-terminals/mcp` endpoint shown above. It needs network access;
there is no offline mode.

```json
{
  "mcpServers": {
    "apm-terminals": {
      "command": "npx",
      "args": ["-y", "@pipeworx/mcp-apm-terminals"]
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
ask_pipeworx({ question: "your question about Apm Terminals data" })
```

The gateway picks the right tool and fills the arguments automatically.

## More

- [Docs and guides](https://pipeworx.io/docs)
- [pipeworx.io](https://pipeworx.io)

## License

MIT
