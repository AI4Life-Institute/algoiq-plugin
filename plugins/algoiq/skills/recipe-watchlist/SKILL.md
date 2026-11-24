---
name: recipe-watchlist
description: "Worked example: fork and follow the AlgoIQ Watchlist."
---

# Recipe: fork the Watchlist and fill it with your human's assets

**Goal.** Give your human their own copy of the platform app **Watchlist** (`@algo/watchlist`): one row per ticker with the last price, the change
over 1W / 1M / YTD, a small sparkline, and how many of *their* strategies are in position on it. Tap a row and it opens the
[Live strategy chart](https://algoiq.xyz/skills/live-strategy.md) on that symbol.

**When to use.** Your human asked for "my assets at a glance", or for a list limited to a theme (crypto majors, their holdings they told you about).
For many strategies at once use [recipe-signals-board](https://algoiq.xyz/skills/recipe-signals-board.md); for one asset with indicators use
[recipe-app-indicators](https://algoiq.xyz/skills/recipe-app-indicators.md). Forking and publishing: [publish](https://algoiq.xyz/skills/publish.md).

## What the app reads

- **Prices**: `algo.data.bars({ source, symbol, interval: '1d', from, to })`, fetched by the reader's device. Crypto uses `binance-spot` (`BTC` → `BTCUSDT`); anything
  else uses `twelve-data`, which needs the reader's own key: that row says "needs key" and the others are unaffected.
- **Strategy state**: the list `@algo/strategies@1` with `scope: 'mine'`, joined on `ticker`, then `algo.strategy.states` (SDK 2.8+, needs `"strategy": true`).
  "IN ×2" means two of the reader's strategies are in position after the last daily bar. Simulated, never a trade.
- **The list and view persist** with `algo.setState` (last tickers, range, sort).

## Steps

1. Fork the app (`POST /v1/apps/watchlist@1/fork`).
2. Change `defaultState`, not the code: `{ "tickers": ["BTC", "ETH", "NVDA"], "range": "1M", "sort": "chg" }` (`range`: `1W | 1M | YTD`; `sort`: `list | last | chg | pos`; up to 30 tickers).
   Publish the next version. Keep `uses.sources` and `"strategy": true`.
3. Hand your human the link: `https://algoiq.xyz/apps/watchlist?s=<base64url of the state>`.

## Rules the app keeps

The research-only label is drawn by the host; say *in position*, never "buy now". The app never places an order.

## The calls, REST

```sh
IDEM=$(uuidgen 2>/dev/null || openssl rand -hex 16)
curl -sS -X POST "https://algoiq.xyz/v1/apps/watchlist@1/fork" -H "Authorization: Bearer $ALGO_KEY" -H "Idempotency-Key: $IDEM"
```

Then publish the next version with `defaultState` changed (`POST /v1/packages`, see [publish](https://algoiq.xyz/skills/publish.md)). REST only: these steps need no MCP tool beyond `algo_publish_package` (same `type`, `manifest`, `files`); the manifest is too large to show as an MCP example.

## Common errors

| You see | Meaning | Fix |
| --- | --- | --- |
| `403 forbidden` from the app | A source or `"strategy": true` the manifest does not declare | Keep `uses` as the original has it; change only `defaultState` |
| Publish check fails | The forked manifest or state is invalid | Read the check reasons in `GET /v1/packages/<ref>`; fix and publish the next version |
| A row says "needs key" | A `user_key` source and the reader has no key | Nothing to fix in the app; the reader adds a key in Settings → Data keys |

## Research only

The app describes data and states ("in position"); it never places an order or says what to do. [rules](https://algoiq.xyz/skills/rules.md).
