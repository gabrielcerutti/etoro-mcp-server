# eToro MCP Server

<div align="center">
  <img src="https://raw.githubusercontent.com/gabrielcerutti/etoro-mcp-server/main/icon.png" alt="eToro MCP" width="100" />

  Connect any MCP-compatible AI assistant (Claude, Cursor, and others) to your eToro account for portfolio analysis, market research, and trading.

  [![CI](https://img.shields.io/github/actions/workflow/status/gabrielcerutti/etoro-mcp-server/ci.yml?branch=main&style=flat-square&label=CI)](https://github.com/gabrielcerutti/etoro-mcp-server/actions/workflows/ci.yml)
  [![Node.js](https://img.shields.io/badge/node-%3E%3D24-brightgreen?style=flat-square)](https://nodejs.org)
  [![TypeScript](https://img.shields.io/badge/TypeScript-5.x-blue?style=flat-square)](https://www.typescriptlang.org)
  [![MCP SDK](https://img.shields.io/badge/MCP%20SDK-1.26%2B-purple?style=flat-square)](https://modelcontextprotocol.io)
  [![License: MIT](https://img.shields.io/badge/license-MIT-orange?style=flat-square)](LICENSE)

  [Overview](#overview) • [Installation](#installation) • [Configuration](#configuration) • [Tools](#tools-35-total)
</div>

---

## Overview

MCP server wrapping the [eToro public API](https://public-api.etoro.com/api/v1), exposing **35 tools** for AI assistants (Claude Desktop, Cursor, Claude Code, etc.). Built for **investment research and portfolio analysis** — assess positions, research instruments and themes, vet popular investors for copy-trading, backtest DCA strategies against historical data. Trading is fully supported, but use care: letting an AI agent place real-money orders is genuinely risky, and you should treat it as such.

## What Can You Do With It?

The server is built around **research workflows** first. The examples below show real multi-tool conversations — the assistant chains several `etoro_*` calls together to answer a single question. Numbers are representative; actual responses draw from live eToro data.

### 1. Portfolio health check

The assistant calls `etoro_get_portfolio` to pull your open positions, then `etoro_get_instruments` to resolve instrument IDs to names and sectors, and `etoro_get_rates` for live prices. It assembles a full snapshot from the raw data.

> **You:** How is my portfolio doing? Break it down by sector and flag any trouble spots.
>
> **Assistant:** I pulled your portfolio (98 open positions), resolved each instrument, and fetched live rates. Here's the picture:
>
> - **Cash:** $22,265 • **Invested:** $45,782 • **Positions:** 98 open
>
> **Sector breakdown:**
>
> | Sector | % of invested | Top holdings |
> | --- | --- | --- |
> | Technology | 38% | GOOG, META, MSFT, NVDA, AMZN |
> | ETFs & Index | 22% | VOO, QQQ, SOXX, ARGT, IEUR |
> | Crypto | 15% | BTC, ETH |
> | Healthcare | 10% | UNH |
> | Financials | 8% | SAP |
> | Other | 7% | LDO + others |
>
> <img src="https://raw.githubusercontent.com/gabrielcerutti/etoro-mcp-server/main/readme-assets/portfolio-allocation.png" width="480" alt="Portfolio sector allocation" />
>
> **Trouble spot:** Your tech sleeve is 38% of invested capital and concentrated in 5 mega-caps. A single sector drawdown hits hard — consider whether the ETF overlap (QQQ, SOXX) is intentional diversification or accidental doubling-down.

**Tools used:** `etoro_get_portfolio` → `etoro_get_instruments` → `etoro_get_rates`

---

### 2. Instrument research

Search is keyword-based — `etoro_search_instruments` finds instruments by ticker or company name, then you chain `etoro_get_instruments` for full details and `etoro_get_candles` for price history.

> **You:** Pull up NVDA — I want the current price and a weekly chart for the last month.
>
> **Assistant:** Searching for "NVDA" and pulling candle data:
>
> - **NVIDIA Corporation** (NVDA) — InstrumentID 1545
> - **Live rate:** Bid $208.14 / Ask $208.40
> - **Exchange:** NASDAQ • **Type:** Stock • **Sector:** Semiconductors
>
> Weekly candles (last 5 weeks):
>
> | Week of | Open | High | Low | Close |
> | --- | --- | --- | --- | --- |
> | Mar 22 | $172.65 | $181.20 | $167.01 | $167.61 |
> | Mar 29 | $165.26 | $177.47 | $162.95 | $176.72 |
> | Apr 5 | $176.12 | $189.98 | $173.66 | $188.53 |
> | Apr 12 | $184.15 | $201.69 | $183.98 | $201.64 |
> | Apr 19 | $199.02 | $210.92 | $197.23 | $208.26 |
>
> <img src="https://raw.githubusercontent.com/gabrielcerutti/etoro-mcp-server/main/readme-assets/nvda-candles.png" width="560" alt="NVDA weekly candlestick chart" />
>
> NVDA rallied +27% off the March low ($162.95 → $208.26), with each weekly candle printing a higher low. Momentum is strong but the move is getting extended.

**Tools used:** `etoro_search_instruments` → `etoro_get_instruments` → `etoro_get_rates` → `etoro_get_candles`

---

### 3. Copy-trading due diligence

Vet popular investors using real public data. The assistant calls `etoro_get_user_profile` for identity and stats, then `etoro_get_user_performance` for the full yearly return history.

> **You:** Vet user jeppekirkbonde for a long-term copy allocation. I want the honest risk picture.
>
> **Assistant:** Here's what the data says on **jeppekirkbonde**:
>
> - **Name:** Jeppe Kirk Bonde • **PI Level:** 6 (highest tier)
> - **Copiers:** 25,000+ • **AUM:** $200M+
> - **Background:** MSc Finance, Copenhagen Business School
>
> **Year-by-year performance (real data):**
>
> | Year | Return | | Year | Return |
> | --- | --- | --- | --- | --- |
> | 2013 | +8.9% | | 2020 | +36.9% |
> | 2014 | −14.3% | | 2021 | +15.9% |
> | 2015 | +22.4% | | 2022 | −19.2% |
> | 2016 | +81.1% | | 2023 | +23.9% |
> | 2017 | +150.9% | | 2024 | +21.0% |
> | 2018 | −10.9% | | 2025 | +37.7% |
> | 2019 | +45.1% | | | |
>
> <img src="https://raw.githubusercontent.com/gabrielcerutti/etoro-mcp-server/main/readme-assets/jeppe-performance.png" width="600" alt="jeppekirkbonde yearly performance" />
>
> **Average annual return:** +30.7% across 13 full years. Three down years (2014, 2018, 2022) with a max drawdown of −19.2% in the 2022 tech selloff.
>
> **What you should know:**
>
> 1. This is a concentrated, long-only US tech strategy. The +150.9% in 2017 and +81.1% in 2016 are outliers — don't expect those every year.
> 2. Down years are real: 2022 cost −19.2%. If you can't stomach a 20% paper loss, this isn't the right copy.
> 3. 25K+ copiers and $200M+ AUM means his moves have market impact in smaller names. Slippage is a factor.
>
> Reasonable for multi-year capital if you accept the volatility. Not appropriate for money you need inside 18 months.

**Tools used:** `etoro_get_user_profile` → `etoro_get_user_performance`

---

### 4. Watchlist management

The full watchlist lifecycle — create, populate, rename, and curate — all through natural language.

> **You:** Create a "Semiconductor" watchlist and add NVDA, AMD, ASML, TSM, and AVGO to it.
>
> **Assistant:** Done:
>
> 1. Created watchlist **"Semiconductor"** (ID: 48291)
> 2. Searched and resolved all 5 tickers to instrument IDs
> 3. Added: NVDA (1545), AMD (2842), ASML (5765), TSM (3901), AVGO (6192)
>
> Your watchlists now:
>
> | Watchlist | Items |
> | --- | --- |
> | Recently Invested | 38 |
> | My Watchlist | 34 |
> | Stocks pre-screening | 69 |
> | ETFs | 54 |
> | Semiconductor ← new | 5 |
> | People & Smart Portfolios | 26 |
> | Crypto | 6 |
> | Commodities | 5 |

**Tools used:** `etoro_create_watchlist` → `etoro_search_instruments` → `etoro_add_watchlist_items` → `etoro_get_watchlists`

---

### 5. DCA backtest with historical candles

Use `etoro_get_candles` to pull years of OHLCV data, then simulate a dollar-cost averaging strategy — no orders placed, pure research.

> **You:** Backtest a $500/month DCA into QQQ over the last 3 years. What would I have today?
>
> **Assistant:** Pulling weekly QQQ candles (156 weeks) and simulating $500 on the first trading day of each month:
>
> - **Total invested:** $18,000 (36 months × $500)
> - **Units accumulated:** ~38.2 QQQ
> - **Current value:** $25,365 (at $664.01/share)
> - **Gain:** +$7,365 (+40.9% on invested capital)
> - **Approx. IRR:** ~12.8%/year
>
> **Worst stretch:** Nov 2022 – Mar 2023, paper drawdown of −22% on the accumulated stack. You bought through it — those discounted shares are now your biggest winners.
>
> No orders placed. This is research only.

**Tools used:** `etoro_get_candles` → `etoro_get_rates` (for current price)

---

### 6. Macro-aware portfolio review

Combine portfolio data with the assistant's general knowledge to get a risk assessment tied to your actual holdings.

> **You:** My portfolio is heavy on tech. Flag any risks from this week's macro calendar.
>
> **Assistant:** Your tech exposure totals ~$17,400 (38% of invested capital) across GOOG, META, MSFT, NVDA, AMZN and the SOXX/QQQ ETFs. This week's watch-list:
>
> - **Wed** — CPI print (core consensus +0.3% MoM). A hot number re-prices rate-cut odds → tech multiples compress first.
> - **Thu** — Fed speakers (Williams, Waller). Hawkish language strengthens USD, pressuring mega-cap international revenue.
> - **Fri** — NVDA earnings (after close). Binary event for your largest single-stock position.
>
> **What this means for you:**
>
> - A hot CPI + hawkish Fed combo could knock 3–5% off your tech sleeve short-term.
> - NVDA earnings will drive ~8% of your portfolio's week-over-week change. Know your thesis before it prints.

**Tools used:** `etoro_get_portfolio` → `etoro_get_instruments` → `etoro_get_rates` (+ assistant's macro knowledge)

---

<details>
<summary><b>Trading is also supported</b> — one example</summary>

> ⚠️
> **Letting an AI execute real-money trades is risky.** The same natural-language interface that makes research fast also makes it easy to misinterpret intent — a typo, an ambiguous number, a stale context from earlier in the conversation. Mitigations: start in `ETORO_TRADING_MODE=demo`, use read-only API keys for anything that doesn't need write access, and require explicit confirmation in your conversation before any `real` order.

All examples above are research-only. When you're ready to execute, the same interface places orders. Orders always respect the configured `ETORO_TRADING_MODE` (`demo` for paper, `real` for live).

> **You:** Buy $500 of NVDA at market.
>
> **Assistant:** Confirmed — opening a market position:
>
> - **Instrument:** NVDA (1545)
> - **Amount:** $500
> - **Rate:** $208.40 (ask)
> - **Mode:** `demo`
>
> Position opened — ID `3418072`. This is a demo trade, no real money involved.

Full trading coverage includes: open position by amount or units (`etoro_open_position`), close or partially close (`etoro_close_position`), limit/entry orders (`etoro_place_limit_order`), cancel pending orders (`etoro_cancel_order`), and listing pending orders (`etoro_get_orders`).

</details>

---

## Composing with Other MCP Servers

MCP servers are designed to be composed. The eToro server handles portfolio data, live rates, candles, and trading — but an AI assistant can call tools from **multiple servers in a single conversation**, filling in the gaps that no single API covers.

### Example: earnings risk check across two servers

With eToro + a financial data server (e.g. [Financial Datasets](https://github.com/financial-datasets/mcp-server), [Alpha Vantage](https://github.com/alphavantage/mcp-server), or similar):

> **You:** Do any of my open positions have earnings this week? Flag the ones I should watch.
>
> **Assistant:** Pulling your portfolio from eToro (98 positions), then checking the earnings calendar:
>
> - **NVDA** — reports Thursday after close. Your position: $4,200 (9.2% of invested). This is a binary event — expect ±8% move.
> - **GOOGL** — reports Tuesday after close. Your position: $3,100 (6.8%). Consensus is +12% YoY revenue; ad revenue mix is the swing factor.
> - **UNH** — reported Monday pre-market (already priced in). Beat on EPS, slight miss on membership growth.
>
> The rest of your holdings have no earnings this week. Your biggest event risk is NVDA Thursday — it's your largest single-stock position.

This isn't possible with the eToro server alone — it knows your positions but not the earnings calendar. The financial data server knows the calendar but not your portfolio. Together, the assistant connects both.

### Natural pairings

| Pair with | What it adds | Example workflow |
| --- | --- | --- |
| **Research & ratings** ([Morningstar](https://developer.morningstar.com/direct-web-services/documentation/morningstar-mcp-server/about)) | Analyst research, fair value estimates, moat ratings, fund data | "What's Morningstar's fair value for my tech holdings?" — pull portfolio, fetch ratings |
| **News & web search** ([Brave Search](https://github.com/brave/brave-search-mcp-server), [Tavily](https://github.com/tavily-ai/tavily-mcp)) | Breaking news, sentiment, event context | "Any news on my holdings today?" — pull portfolio, search each ticker |
| **Financial data** ([Financial Datasets](https://github.com/financial-datasets/mcp-server), [Alpha Vantage](https://mcp.alphavantage.co/)) | Fundamentals, earnings, SEC filings, economic calendar | "Show me P/E ratios for my tech positions" — pull holdings, fetch fundamentals |
| **Market data** ([Polygon.io](https://github.com/polygon-io/mcp_polygon), [Twelve Data](https://github.com/twelvedata/mcp), [Massive](https://github.com/massive-com/mcp_massive)) | Real-time streaming, options chains, screener universes | "Screen the S&P 500 for momentum setups, then add the top 5 to my watchlist" |
| **Free alternative** ([Yahoo Finance](https://github.com/Alex2Yang97/yahoo-finance-mcp)) | Prices, financials, options, market news (no API key needed) | "Compare earnings growth for NVDA vs AMD over the last 4 quarters" |

### Configuration

Just add multiple servers to your MCP config — the assistant sees all tools from all servers:

```json
{
  "mcpServers": {
    "etoro": {
      "command": "npx",
      "args": ["-y", "etoro-mcp-server"],
      "env": {
        "ETORO_API_KEY": "your-api-key",
        "ETORO_USER_KEY": "your-user-key",
        "ETORO_TRADING_MODE": "demo"
      }
    },
    "financial-data": {
      "command": "npx",
      "args": ["-y", "financial-datasets-mcp-server"],
      "env": {
        "FINANCIAL_DATASETS_API_KEY": "your-key"
      }
    },
    "news": {
      "command": "npx",
      "args": ["-y", "tavily-mcp-server"],
      "env": {
        "TAVILY_API_KEY": "your-key"
      }
    }
  }
}
```

The assistant will automatically chain tools across servers when a question requires it — no extra wiring needed.

---

## Installation

### Claude Desktop (recommended)

Download the latest `.mcpb` file from the [Releases](../../releases) page and drag it into Claude Desktop:

**Extensions → drag `etoro-{version}.mcpb` into the window**

You'll be prompted for your API Key, User Key, and trading mode. That's it.

> [!TIP]
> Get your keys at **eToro → Settings → Trading → Create New Key**. Choose your environment (Demo or Real) and the permissions you need (Read or Write).

---

### Via npm (Claude Code, Cursor, Continue, Cody, …)

Add this to your client's MCP config — it'll launch the server from the published npm package on demand, no local install needed:

```json
{
  "mcpServers": {
    "etoro": {
      "command": "npx",
      "args": ["-y", "etoro-mcp-server"],
      "env": {
        "ETORO_API_KEY": "your-api-key",
        "ETORO_USER_KEY": "your-user-key",
        "ETORO_TRADING_MODE": "demo"
      }
    }
  }
}
```

For Claude Code, the equivalent CLI command:

```bash
claude mcp add etoro \
  -e ETORO_API_KEY=your-api-key \
  -e ETORO_USER_KEY=your-user-key \
  -e ETORO_TRADING_MODE=demo \
  -- npx -y etoro-mcp-server
```

Pin a specific version with `etoro-mcp-server@1.1.0` in the args if you want stability over auto-updates.

---

### From source (developers)

If you want to hack on the server locally, clone the repo and build:

```bash
npm install
npm run build
```

Then point your MCP client at the built file (`dist/index.js`):

```bash
claude mcp add etoro-mcp \
  -e ETORO_API_KEY=your-api-key \
  -e ETORO_USER_KEY=your-user-key \
  -e ETORO_TRADING_MODE=demo \
  -- node /path/to/etoro-mcp/dist/index.js
```

Or in `claude_desktop_config.json` / equivalent:

```json
{
  "mcpServers": {
    "etoro-mcp": {
      "command": "node",
      "args": ["/path/to/etoro-mcp/dist/index.js"],
      "env": {
        "ETORO_API_KEY": "your-api-key",
        "ETORO_USER_KEY": "your-user-key",
        "ETORO_TRADING_MODE": "demo"
      }
    }
  }
}
```

---

## MCP Registry

This server is listed on the [MCP Registry](https://registry.modelcontextprotocol.io) under the name [`io.github.gabrielcerutti/etoro-mcp-server`](https://registry.modelcontextprotocol.io/v0/servers?search=etoro-mcp-server). The registry is the public index of MCP servers — think of it as a package registry for AI tools, analogous to what npm is for JavaScript libraries. MCP-compatible clients use it to discover and install servers programmatically without users having to hand-copy configuration.

Each tagged release of this repository is auto-published to the registry by GitHub Actions (via OIDC), so the entry always tracks the latest npm version.

---

## Configuration

| Setting | Env var | CLI arg | Default |
| --- | --- | --- | --- |
| API Key | `ETORO_API_KEY` | `--api-key` | (none) |
| User Key | `ETORO_USER_KEY` | `--user-key` | (none) |
| Trading Mode | `ETORO_TRADING_MODE` | `--trading-mode` | `demo` |

**Trading mode:** `demo` routes all trading calls through eToro's virtual account. Set to `real` only when you're ready to trade with real money.

> [!WARNING]
> `trading_mode` must match the environment of your API key. A demo key used with `real` mode (or vice versa) will cause authentication errors.

---

## Tools (35 total)

All tools are prefixed with `etoro_` for namespace isolation when composed with other MCP servers.

### Market Data (8)

| Tool | Description |
| --- | --- |
| `etoro_search_instruments` | Search instruments by keyword (e.g. "AAPL", "Bitcoin") or exact ticker |
| `etoro_get_instruments` | Get full instrument details by IDs (1–100) |
| `etoro_get_instrument_types` | List all instrument types (stocks, crypto, ETFs…) |
| `etoro_get_industries` | List industry classifications |
| `etoro_get_exchanges` | List stock exchanges |
| `etoro_get_candles` | Get OHLCV candle data for technical analysis and backtesting |
| `etoro_get_closing_prices` | Get historical daily closing prices |
| `etoro_get_rates` | Get live bid/ask rates |

### Portfolio & Trading (7)

| Tool | Description |
| --- | --- |
| `etoro_get_portfolio` | Get portfolio + P&L: positions, unrealized P&L, exposure summary, longest holding |
| `etoro_get_orders` | List all pending orders (limit / entry) |
| `etoro_get_trade_history` | Closed-trade history (entry/exit, P&L, duration). Primary source for performance research |
| `etoro_open_position` | Open position by USD amount **or** units (unified tool) |
| `etoro_close_position` | Close an open position fully, or partially via `unitsToDeduct` |
| `etoro_place_limit_order` | Place a limit / entry order |
| `etoro_cancel_order` | Cancel a pending order |

### User & Discovery (7)

| Tool | Description |
| --- | --- |
| `etoro_get_current_user` | Get the authenticated user's identity (GCID, real / demo CIDs) |
| `etoro_get_user_profile` | Get a user's public profile |
| `etoro_get_user_performance` | Get performance summary (optionally by time period) |
| `etoro_get_user_trades` | Get a user's trade info for a period |
| `etoro_get_user_portfolio` | Get a user's live public portfolio holdings |
| `etoro_discover_users` | Discover popular investors filtered by gain, risk score, period |
| `etoro_get_copiers` | Get info about users copying your portfolio |

### Watchlists (9)

| Tool | Description |
| --- | --- |
| `etoro_get_watchlists` | List your watchlists |
| `etoro_create_watchlist` | Create a watchlist |
| `etoro_delete_watchlist` | Delete a watchlist |
| `etoro_rename_watchlist` | Rename a watchlist |
| `etoro_add_watchlist_items` | Add instruments to a watchlist |
| `etoro_remove_watchlist_item` | Remove an instrument from a watchlist |
| `etoro_set_default_watchlist` | Set default watchlist |
| `etoro_get_curated_lists` | Get eToro's curated lists |
| `etoro_get_public_watchlists` | Browse a user's public watchlists |

### Social Feeds (4)

| Tool | Description |
| --- | --- |
| `etoro_get_instrument_feed` | Get social feed for an instrument |
| `etoro_get_user_feed` | Get social feed for a user |
| `etoro_create_post` | Create a social feed post |
| `etoro_create_comment` | Comment on a post |

---

## Disclaimer

This project is an **unofficial**, community-built integration. It is not affiliated with, endorsed by, or supported by eToro. Use it at your own risk.

- **Not financial advice.** Nothing in this server, its documentation, or its example outputs constitutes investment advice, a recommendation, or a solicitation to buy or sell any financial instrument.
- **You are responsible for your trades.** The server can place real-money orders when configured in `real` mode. Verify every order before confirming, and understand that AI assistants can misinterpret intent.
- **API changes may break things.** This server depends on eToro's public API, which can change without notice. Always pin a version in production.
- **Demo first.** Start with `ETORO_TRADING_MODE=demo` and only switch to `real` after you're confident in your setup.

---

## Contributing

Contributions are welcome — bug reports, feature requests, and pull requests.

1. Fork the repo and create a branch from `main`
2. Run `npm install && npm run build` to verify your changes compile
3. Run `npm test` to make sure existing tests pass
4. Open a PR with a clear description of what changed and why

---

## License

[MIT](LICENSE) — use it, fork it, build on it.