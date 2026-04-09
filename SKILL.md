---
name: coinank-openapi
description: >
  Fetches cryptocurrency market data — including prices, funding rates, open interest, liquidations, long/short ratios, trading volume, and market cap — from the CoinAnk API.
  Use when the user asks about crypto prices, BTC or ETH market data, funding rates, liquidation heatmaps, buy/sell volume, open interest, token metrics, or mentions CoinAnk, Bitcoin, Ethereum, altcoin, or any on-chain / derivatives market statistics.
metadata:
  openclaw:
    homepage: "https://coinank.com"
    requires:
      env:
        - COINANK_API_KEY
    primaryEnv: COINANK_API_KEY
    priority: 100
    keywords:
      - bitcoin
      - btc
      - ethereum
      - eth
      - cryptocurrency
      - crypto
      - altcoin
      - token
      - price
      - market cap
      - trading volume
      - funding rate
      - open interest
      - liquidation
      - long short ratio
      - buy sell volume
      - derivatives
      - 价格
      - 走势
      - 爆仓
      - 多空比
      - 资金费率
      - 未平仓量
---

# SECURITY MANIFEST
# - Allowed to read: {baseDir}/README.md, {baseDir}/references/*.json
# - Allowed to make network requests to: https://open-api.coinank.com

## Quick Setup

Set your API key before use:
```bash
export COINANK_API_KEY="your_api_key"
```

## Workflow (On-Demand Loading)

When the user makes a request, follow these steps in order:

1. **Check API key** — Verify `COINANK_API_KEY` exists in the environment. If missing, prompt the user: `export COINANK_API_KEY="your_api_key"` and stop.
2. **Read README** — Read `{baseDir}/README.md` for an overview of available endpoints and authentication details.
3. **Index references** — List filenames in `{baseDir}/references/`. Identify which OpenAPI JSON files match the user's request (e.g., funding rate → `funding_rate.json`). Do NOT load all files at once.
4. **Read only relevant JSON** — Load only the selected `.json` file(s). In each file, `paths` is an object whose keys are the API paths. Inspect `paths`, `parameters`, and `requestBody`.
5. **Build and run the request** — Execute with `curl`:
   - **Base URL**: `https://open-api.coinank.com` (or from the JSON `servers` field)
   - **Auth**: inject the API key as a header — `-H "apikey: $COINANK_API_KEY"`
   - **Timestamps**: always use the current millisecond timestamp (see below); JSON examples contain stale historical values

### Example request

```bash
# Get current millisecond timestamp (cross-platform)
NOW=$(python3 -c "import time; print(int(time.time()*1000))")

# Example: fetch BTC funding rate history
curl -s "https://open-api.coinank.com/api/fundingRate/history?symbol=BTCUSDT&interval=1h&endTime=$NOW" \
  -H "apikey: $COINANK_API_KEY" | python3 -m json.tool
```

## ⚠️ Critical Rules

| Rule | Detail |
|------|--------|
| **No bulk loading** | Load only JSON files relevant to the current request — never all at once |
| **Validate params first** | Check all required parameters against the OpenAPI definition before sending |
| **Timestamps** | Use `python3 -c "import time; print(int(time.time()*1000))"` — `date +%s%3N` is broken on macOS |
| **Extra params** | Some endpoints (e.g., `getLiqHeatMap`) reject unknown params like `endTime` or `size` — send only what the spec lists |
| **`exchanges` param** | Required for aggregate endpoints (`getAggCvd`, `getAggBuySellCount`, etc.) — pass `exchanges=` (empty string) to aggregate all exchanges |
| **Error handling** | On failure, show the user a friendly message and log the full error response |

## interval Values by Endpoint Type

| Endpoint type | Accepted interval values |
|---------------|--------------------------|
| Candlestick / market orders / long-short ratio / OI | `1m 3m 5m 15m 30m 1h 2h 4h 6h 8h 12h 1d` |
| Liquidation heatmap (`getLiqHeatMap`) | `12h 1d 3d 1w 2w 1M 3M 6M 1Y` |
| RSI screener | `1H 4H 1D` (uppercase) |
| Funding rate heatmap | `1D 1W 1M 6M` |

Always check the `description` field of each parameter in the relevant `.json` file for the authoritative list.

## Response Format

Success is indicated by `"code": "1"` (string `"1"`, not integer). Some endpoints return a nested `data` structure:

```json
{"success": true, "code": "1", "data": {"success": true, "code": "1", "data": [...]}}
```

Check the actual type of `data` before parsing and unwrap the inner layer as needed.
