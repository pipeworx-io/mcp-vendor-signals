# Vendor Signals — what your accounts just shipped

Changelog entries, release notes and product announcements for **31 tracked software vendors**, dated and linked, joinable by domain against a target list.

Part of [Pipeworx](https://pipeworx.io) — an MCP gateway connecting AI agents to 1857+ live data sources. This is an independent, unofficial integration — not affiliated with, endorsed by, or published by the upstream provider.

## The GTM read

"This account just shipped X" is a reason to reach out. That is the unit here — a dated, sourced change attached to a vendor — not developer news, which `dev-feeds` already carries.

## Tools

| Tool | Answers |
|---|---|
| `vendor_recent_changes` | *What did Stripe ship this month?* |
| `vendor_signal_sweep` | *Which of my accounts were active this week?* |
| `vendor_coverage` | Which vendors are tracked, and is each feed healthy? |

## Auth

Keyless.

## The distinction this pack is built around

**"Tracked but quiet" and "not tracked" are different answers**, and a seller acts differently on each. Returning an empty list for both would make an unwatched account look like a dormant one.

- Tracked and quiet → `found: true, count: 0` plus a note saying it is a real quiet period.
- Not tracked → `found: false, reason: "vendor_not_tracked"` and an explicit statement that this is missing **coverage**, not missing **activity**.
- Feed currently failing → flagged on the response, because an empty result then means *unknown*, not *inactive*.

`vendor_signal_sweep` returns `quiet_vendors` for the same reason: a vendor in neither list is not tracked at all.

## Scope

Structured publisher feeds only — changelog RSS/Atom and GitHub releases. Deliberately **not** a general web-diffing crawler. Titles, summaries and links are stored; **full page content is never retained**.

Vendor list: `vendor_doc_sources` (seeded by `scripts/ingest-vendor-signals.mjs --seed`). Schema: `supabase/migrations/*_vendor_signals.sql`.

### Things the next person would otherwise rediscover

- **A dead feed does not error — it returns zero entries and looks healthy.** Slack's changelog feed answers HTTP 200 with `text/html` and no entries; six of thirty candidates behaved this way (Cloudflare, Linear, Twilio, Datadog, Atlassian, Slack on their first-guess URLs). Every candidate was probed before seeding, and the poller treats **zero parseable entries as a failure**, not a quiet success, so a rotted feed surfaces in `vendor_coverage` instead of silently reporting "no changes" forever.
- **GitHub `releases.atom` is the reliable fallback** when a vendor has no changelog feed — `https://github.com/<org>/<repo>/releases.atom` works for most vendors with a public SDK, and is how Twilio, Linear, Datadog and Slack are covered here.
- **Vercel's feed returns ~1,500 entries** where most return 10. Any per-source assumption of a small page is wrong.
- **`Prefer: return=minimal` answers 201 with an empty body**, not 204 — code that switches on 204 alone will try to parse nothing.

## Quick Start

Add to your MCP client (Claude Desktop, Cursor, Windsurf, etc.):

```json
{
  "mcpServers": {
    "vendor-signals": {
      "url": "https://gateway.pipeworx.io/vendor-signals/mcp"
    }
  }
}
```

### What this endpoint actually serves

`tools/list` at `https://gateway.pipeworx.io/vendor-signals/mcp` returns the tools in the table
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

```bash
curl -X POST https://gateway.pipeworx.io/v1/tools/vendor_recent_changes \
  -H 'Content-Type: application/json' \
  -d '{"vendor":"stripe","days":30}'
```

No account needed for the first calls. Inspect any tool: `GET https://gateway.pipeworx.io/v1/tools/vendor_recent_changes`. Find one: `POST https://gateway.pipeworx.io/v1/tools/search_packs` with `{"query":"..."}`.

## Run it from npm

`@pipeworx/mcp-vendor-signals` is a local stdio bridge to the Pipeworx gateway: your MCP client
starts it with `npx`, and it forwards every call to the same
`https://gateway.pipeworx.io/vendor-signals/mcp` endpoint shown above. It needs network access;
there is no offline mode.

```json
{
  "mcpServers": {
    "vendor-signals": {
      "command": "npx",
      "args": ["-y", "@pipeworx/mcp-vendor-signals"]
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
ask_pipeworx({ question: "your question about Vendor Signals data" })
```

The gateway picks the right tool and fills the arguments automatically.

## More

- [Docs and guides](https://pipeworx.io/docs)
- [pipeworx.io](https://pipeworx.io)

## License

MIT
