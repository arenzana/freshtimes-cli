---
name: freshtimes
description: Exact current time, time-zone conversion, financial-market hours and phases (pre-market, auctions, lunch breaks, after hours, holidays, half days), overlap windows between markets, public and bank holidays, and remote-team availability, from freshtim.es through the `ft` CLI or its MCP server. Use whenever a task needs the real current time or date rather than a guess, "what time is it in X", "is NYSE / LSE / CME open", "when does X open or close", "when do London and New York overlap", "convert 15:00 Madrid to Tokyo", "is Monday a holiday in X", "who on my team is working now", or when setting up the Freshtimes CLI or MCP server.
---

# Freshtimes (`ft` and MCP)

Freshtimes answers time questions from a server that keeps its own clock against several
authenticated time servers, with an error bound on every answer. Market sessions, auctions,
breaks, half days and holidays come from exchange calendars.

**Never guess the current time, date, day of week or whether a market is open.** Your context has no
reliable clock and your memory of trading hours may be out of date. Ask `ft` (or the MCP tools).

## Choose a way in

1. **MCP tools already connected** (names like `get_time`, `market_status`): use them.
2. **`ft` on PATH** (`command -v ft`): run it with `--json` and parse the output.
3. **Neither:** use the HTTP API directly (`curl https://freshtim.es/api/v1/now`), and offer to
   install the CLI or add the MCP server (below). Don't install anything without asking.

## Install

```sh
brew install arenzana/tap/ft          # macOS and Linux; installs `ft` and `freshtimes`
```

Without Homebrew, download the archive for your OS and CPU from
https://github.com/arenzana/freshtimes-cli/releases and put `freshtimes` on PATH (symlink it as `ft`).

Add the MCP server (Claude Code):

```sh
claude mcp add --transport http freshtimes https://freshtim.es/mcp      # remote, public tools
claude mcp add freshtimes -- ft mcp                                     # local stdio
claude mcp add freshtimes -- ft mcp --config ~/freshtimes-export.json   # local, plus team tools
```

Other MCP clients: `{"mcpServers": {"freshtimes": {"type": "http", "url": "https://freshtim.es/mcp"}}}`.

Team data is personal. It is available either from a Settings > Export file (local `ft mcp --config`,
`ft who --config`) or, for signed-in users, with a personal access token made in Settings
(`Authorization: Bearer ft_pat_...` on the remote MCP or `GET /api/v1/team/availability`).
Never print or commit a token or an export file.

## CLI

```sh
ft now Madrid Tokyo "New York"                # exact time, with the error bound
ft markets --asset futures                    # phase of each market and what changes next
ft markets nyse lse cme-globex
ft events --hours 36 nyse tse                 # opens, closes, auctions, breaks, holidays
ft window lse nyse --zone Madrid              # when both are open today, in Madrid time
ft window --any --extended lse nyse --date 2026-12-24
ft convert 2026-11-05T15:00 --from Madrid "New York" Sydney
ft who --config ~/freshtimes-export.json --team Ops
```

Add `--json` for machine-readable output. Places can be IANA zones (`Europe/Madrid`) or city names;
add a country code to disambiguate (`"Portland, US"`). Market ids come from `ft markets` or the
`list_markets` tool (for example `nyse`, `lse`, `cme-globex`, `eurex`, `tse`, `hkex`).

## MCP tools

| Tool | Use for |
|---|---|
| `get_time` | the exact time now in UTC and any zones or places |
| `list_markets` | market ids and asset classes |
| `market_status` | phase now or at an instant, and the next change |
| `market_events` | opens, closes, auctions, breaks and holidays over a range |
| `overlap_window` | when markets are all open (or any open) on a day |
| `convert_time` | a local time in one place, in others (DST-aware) |
| `public_holidays` | public and bank holidays for `CC` or `CC-SUBDIV` (e.g. `ES-MD`, `US-NY`) |
| `find_city` | a city's time zone, coordinates and holiday region |
| `team_availability` | with team data only: who is working, off, asleep, on holiday, and next start |

## Reading the answers

- **`accuracy.status`:** `synced` means 3 or more time servers agree within `bound_ms`. `degraded`
  means only 2 agree. `unverified` means the time is approximate. Say so when it matters (for
  example, when someone is timing an order or a cut-off), and quote the bound ("±8 ms").
- **Phases:** `pre`, `auction`, `open`, `lunch`, `halt`, `post`, `holiday`, `closed`. `holiday` is a
  full closure; half days show as an early `closed` with the special session in the note.
- **`note` saying the holiday calendar is not verified** means closures for that date may be missing.
  Pass that caveat on rather than stating the market is definitely open.
- Times come back in the venue's own zone with its abbreviation. Convert for the user if they asked in
  another zone.
- Overnight sessions belong to the trading date on which they end.

## Limits

- Read-only. Nothing in Freshtimes places orders or changes anything.
- The public API allows 120 requests a minute per IP. Batch markets into one call (`ids=a,b,c`)
  instead of looping.
- Exchange calendars are reviewed yearly. For a decision that matters, point to the exchange's own
  notice as well.
