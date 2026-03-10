<div align="center">

# CoinAnk OpenAPI Skill

### `> Crypto derivatives data engine for AI agents_`

<br />

[![Endpoints](https://img.shields.io/badge/59_Endpoints-18_Categories-00d4aa?style=for-the-badge&logo=bitcoin&logoColor=white)](#api-overview)
[![OpenClaw](https://img.shields.io/badge/OpenClaw-Skill-ff6b6b?style=for-the-badge&logo=openai&logoColor=white)](https://github.com/openclaw/openclaw)
[![REST](https://img.shields.io/badge/REST-API-3178c6?style=for-the-badge&logo=fastapi&logoColor=white)](https://open-api.coinank.com)
[![License](https://img.shields.io/badge/MIT-License-f59e0b?style=for-the-badge&logo=opensourceinitiative&logoColor=white)](./LICENSE)

<br />

[简体中文](./README.md) · [English](./README_EN.md)

<br />

<img src="https://raw.githubusercontent.com/andreasbm/readme/master/assets/lines/rainbow.png" alt="-----" />

</div>

<div align="center">

## What is CoinAnk OpenAPI Skill?

**One sentence to query data. 18 categories of derivatives indicators at your fingertips.**

</div>

<div align="center">

CoinAnk OpenAPI Skill is an [OpenClaw](https://github.com/openclaw/openclaw) Skill (AI agent plugin) that provides LLMs with comprehensive crypto derivatives market data capabilities. It covers **K-lines, ETF, open interest, long/short ratios, funding rates, liquidations, order flow, whale movements** and more — 18 categories, 59 real-time data endpoints, all battle-tested and verified.

</div>

<div align="center">

<table>
<tr><td>

- All **59 endpoints** tested and verified
- All requests are **GET** — simple and efficient
- Supports **VIP1 ~ VIP4** tiered access levels

</td></tr>
</table>

<br />
<img src="https://raw.githubusercontent.com/andreasbm/readme/master/assets/lines/rainbow.png" alt="-----" />

## Data Coverage

</div>

<div align="center">
<table>
<tr>
<td width="50%">

**Market & Price**

| Category | Endpoints | Tier |
|:--|:--:|:--:|
| K-Lines | 1 | VIP1 |
| ETF | 5 | VIP1 |
| Coins & Pairs | 4 | VIP1 |
| Indicators | 10 | VIP1 |
| News & Flash | 2 | VIP2 |

</td>
<td width="50%">

**Derivatives Depth**

| Category | Endpoints | Tier |
|:--|:--:|:--:|
| Open Interest | 7 | VIP1 |
| Funding Rates | 7 | VIP1 |
| Long/Short Ratio | 6 | VIP1 |
| Liquidation | 8 | VIP1 |
| RSI Screener | 1 | VIP2 |

</td>
</tr>
<tr>
<td width="50%">

**Institutional-Grade**

| Category | Endpoints | Tier |
|:--|:--:|:--:|
| Large Orders | 2 | VIP3 |
| Market Order Stats | 8 | VIP3 |
| Order Book | 3 | VIP3 |
| Fund Flow | 2 | VIP3 |
| Order Flow | 1 | VIP3 |
| Net Long/Short | 1 | VIP3 |

</td>
<td width="50%">

**On-Chain & Whales**

| Category | Endpoints | Tier |
|:--|:--:|:--:|
| HyperLiquid Whales | 2 | VIP2 |
| Trending Rankings | 8 | VIP2 |

</td>
</tr>
</table>
</div>

<div align="center">

**Total: 18 categories · 59 endpoints**

<br />
<img src="https://raw.githubusercontent.com/andreasbm/readme/master/assets/lines/rainbow.png" alt="-----" />

## Quick Start

</div>

```bash
# 1. Clone into your OpenClaw skills directory
git clone https://github.com/coinank/coinank-openapi-skill.git ~/.openclaw/skills/coinank-openapi-skill

# 2. Set your API key as an environment variable
export COINANK_API_KEY="your_api_key_here"
```

<div align="center">

Then query directly with natural language in your OpenClaw agent:

</div>

```
> "Check the current BTC funding rate"
> "Show me the liquidation data for the past 24 hours"
> "Get the ETH long/short ratio"
```

<div align="center">
<br />
<img src="https://raw.githubusercontent.com/andreasbm/readme/master/assets/lines/rainbow.png" alt="-----" />

## Authentication & Request Spec

</div>

| Item | Description |
|------|-------------|
| **Base URL** | `https://open-api.coinank.com` |
| **Auth** | HTTP Header: `apikey: <your_api_key>` |
| **Method** | All `GET` |
| **Response** | `application/json` |
| **Success** | `{"success": true, "code": "1", "data": ...}` |

### Standard Response

```json
{
  "success": true,
  "code": "1",
  "data": [ ... ]
}
```

### Error Codes

| code | Meaning |
|------|---------|
| `1` | Success |
| `-3` | Invalid API key or auth failure |
| `-7` | Time range exceeded (bad endTime) |
| `0` | System error (missing params or server error) |

<div align="center">
<br />
<img src="https://raw.githubusercontent.com/andreasbm/readme/master/assets/lines/rainbow.png" alt="-----" />

## Important Notes

</div>

### 1. Timestamps Must Be Milliseconds & Current

All `endTime` parameters require **millisecond timestamps** close to the current time. Expired or malformed timestamps return `code: -7`.

```bash
# Correct: use python3 (cross-platform)
NOW=$(python3 -c "import time; print(int(time.time()*1000))")

# Wrong: macOS date doesn't support %3N, produces invalid values like "17228693N"
NOW=$(date +%s%3N)  # Don't use this!
```

### 2. VIP Tier System

Endpoints are divided into VIP1–VIP4 tiers. Higher tiers unlock more endpoints. Each endpoint indicates its minimum required tier.

### 3. `exchanges` Parameter Must Be Passed

For aggregate market order endpoints (`getAggCvd`, `getAggBuySellCount`, etc.), the `exchanges` parameter is **required** (pass an empty string `exchanges=` to aggregate all exchanges).

### 4. Example Timestamps Are Historical

Timestamps in the `references/` directory JSON files are historical examples only. Always use real-time generated timestamps when making calls.

<div align="center">
<br />
<img src="https://raw.githubusercontent.com/andreasbm/readme/master/assets/lines/rainbow.png" alt="-----" />

## API Details

</div>

---

<details>
<summary><strong>1. K-Lines</strong> — 1 endpoint · VIP1</summary>

<br />

#### `GET /api/kline/lists` — K-Line Market Data

| Param | Required | Type | Description | Example |
|-------|----------|------|-------------|---------|
| `symbol` | Yes | string | Trading pair | `BTCUSDT` |
| `exchange` | Yes | string | Exchange | `Binance` |
| `endTime` | Yes | number | Millisecond timestamp | `current timestamp` |
| `size` | Yes | integer | Count, max 500 | `10` |
| `interval` | Yes | string | Period, see enums | `1h` |
| `productType` | Yes | string | `SWAP` futures / `SPOT` spot | `SWAP` |

```bash
curl -H "apikey: $APIKEY" \
  "https://open-api.coinank.com/api/kline/lists?symbol=BTCUSDT&exchange=Binance&endTime=$NOW&size=10&interval=1h&productType=SWAP"
```

</details>

---

<details>
<summary><strong>2. ETF</strong> — 5 endpoints · VIP1</summary>

<br />

#### `GET /api/etf/getUsBtcEtf` — US BTC ETF List
**No parameters required**

#### `GET /api/etf/getUsEthEtf` — US ETH ETF List
**No parameters required**

#### `GET /api/etf/usBtcInflow` — US BTC ETF Historical Net Inflow
**No parameters required**

#### `GET /api/etf/usEthInflow` — US ETH ETF Historical Net Inflow
**No parameters required**

#### `GET /api/etf/hkEtfInflow` — HK ETF Historical Net Inflow
**No parameters required**

```bash
curl -H "apikey: $APIKEY" "https://open-api.coinank.com/api/etf/getUsBtcEtf"
curl -H "apikey: $APIKEY" "https://open-api.coinank.com/api/etf/getUsEthEtf"
curl -H "apikey: $APIKEY" "https://open-api.coinank.com/api/etf/usBtcInflow"
curl -H "apikey: $APIKEY" "https://open-api.coinank.com/api/etf/usEthInflow"
curl -H "apikey: $APIKEY" "https://open-api.coinank.com/api/etf/hkEtfInflow"
```

</details>

---

<details>
<summary><strong>3. HyperLiquid Whales</strong> — 2 endpoints · VIP2</summary>

<br />

#### `GET /api/hyper/topPosition` — Whale Position Rankings

| Param | Required | Type | Description | Example |
|-------|----------|------|-------------|---------|
| `sortBy` | Yes | string | Sort field | `positionValue` |
| `sortType` | Yes | string | `desc` / `asc` | `desc` |
| `page` | Yes | integer | Page number | `1` |
| `size` | Yes | integer | Page size | `10` |

#### `GET /api/hyper/topAction` — Latest Whale Activity
**No parameters required**

```bash
curl -H "apikey: $APIKEY" \
  "https://open-api.coinank.com/api/hyper/topPosition?sortBy=positionValue&sortType=desc&page=1&size=10"

curl -H "apikey: $APIKEY" "https://open-api.coinank.com/api/hyper/topAction"
```

</details>

---

<details>
<summary><strong>4. Net Long & Short</strong> — 1 endpoint · VIP3</summary>

<br />

#### `GET /api/netPositions/getNetPositions` — Net Long/Short History

| Param | Required | Type | Description | Example |
|-------|----------|------|-------------|---------|
| `exchange` | Yes | string | Exchange | `Binance` |
| `symbol` | Yes | string | Trading pair | `BTCUSDT` |
| `interval` | Yes | string | Period | `1h` |
| `endTime` | Yes | number | Millisecond timestamp | `current timestamp` |
| `size` | Yes | integer | Count, max 500 | `10` |

```bash
curl -H "apikey: $APIKEY" \
  "https://open-api.coinank.com/api/netPositions/getNetPositions?exchange=Binance&symbol=BTCUSDT&interval=1h&endTime=$NOW&size=10"
```

</details>

---

<details>
<summary><strong>5. Large Orders</strong> — 2 endpoints · VIP3</summary>

<br />

#### `GET /api/trades/largeTrades` — Large Market Orders

| Param | Required | Type | Description | Example |
|-------|----------|------|-------------|---------|
| `symbol` | Yes | string | Trading pair | `BTCUSDT` |
| `productType` | Yes | string | `SWAP` / `SPOT` | `SWAP` |
| `amount` | Yes | string | Min amount (USD) | `10000000` |
| `endTime` | Yes | string | Millisecond timestamp | `current timestamp` |
| `size` | Yes | string | Count, max 500 | `10` |

#### `GET /api/bigOrder/queryOrderList` — Large Limit Orders

| Param | Required | Type | Description | Example |
|-------|----------|------|-------------|---------|
| `symbol` | Yes | string | Trading pair | `BTCUSDT` |
| `exchangeType` | Yes | string | `SWAP` / `SPOT` / `FUTURES` | `SWAP` |
| `size` | Yes | integer | Count, max 500 | `10` |
| `amount` | Yes | number | Min amount (USD) | `1000000` |
| `side` | Yes | string | `ask` sell / `bid` buy | `ask` |
| `exchange` | Yes | string | Exchange (Binance / OKX / Coinbase) | `Binance` |
| `isHistory` | Yes | string | `true` historical / `false` real-time | `true` |
| `startTime` | No | number | End timestamp (recommended for isHistory=true) | `current timestamp` |

```bash
curl -H "apikey: $APIKEY" \
  "https://open-api.coinank.com/api/trades/largeTrades?symbol=BTCUSDT&productType=SWAP&amount=10000000&endTime=$NOW&size=10"

curl -H "apikey: $APIKEY" \
  "https://open-api.coinank.com/api/bigOrder/queryOrderList?symbol=BTCUSDT&exchangeType=SWAP&size=10&amount=1000000&side=ask&exchange=Binance&isHistory=true&startTime=$NOW"
```

</details>

---

<details>
<summary><strong>6. Coins & Trading Pairs</strong> — 4 endpoints · VIP1</summary>

<br />

#### `GET /api/instruments/getLastPrice` — Real-Time Price

| Param | Required | Type | Description | Example |
|-------|----------|------|-------------|---------|
| `symbol` | Yes | string | Trading pair | `BTCUSDT` |
| `exchange` | Yes | string | Exchange | `Binance` |
| `productType` | Yes | string | `SWAP` / `SPOT` | `SWAP` |

#### `GET /api/instruments/getCoinMarketCap` — Coin Market Cap Info

| Param | Required | Type | Description | Example |
|-------|----------|------|-------------|---------|
| `baseCoin` | Yes | string | Coin | `BTC` |

#### `GET /api/baseCoin/list` — Supported Coins List

| Param | Required | Type | Description | Example |
|-------|----------|------|-------------|---------|
| `productType` | Yes | string | `SWAP` / `SPOT` | `SWAP` |

#### `GET /api/baseCoin/symbols` — Supported Trading Pairs

| Param | Required | Type | Description | Example |
|-------|----------|------|-------------|---------|
| `exchange` | Yes | string | Exchange | `Binance` |
| `productType` | Yes | string | `SWAP` / `SPOT` | `SWAP` |

```bash
curl -H "apikey: $APIKEY" \
  "https://open-api.coinank.com/api/instruments/getLastPrice?symbol=BTCUSDT&exchange=Binance&productType=SWAP"

curl -H "apikey: $APIKEY" \
  "https://open-api.coinank.com/api/instruments/getCoinMarketCap?baseCoin=BTC"

curl -H "apikey: $APIKEY" \
  "https://open-api.coinank.com/api/baseCoin/list?productType=SWAP"

curl -H "apikey: $APIKEY" \
  "https://open-api.coinank.com/api/baseCoin/symbols?exchange=Binance&productType=SWAP"
```

</details>

---

<details>
<summary><strong>7. Long/Short Ratio</strong> — 6 endpoints · VIP1</summary>

<br />

#### `GET /api/longshort/buySell` — Global Long/Short Buy/Sell Ratio
**Tier: VIP3**

| Param | Required | Type | Description | Example |
|-------|----------|------|-------------|---------|
| `baseCoin` | Yes | string | Coin | `BTC` |
| `interval` | Yes | string | Period | `1h` |
| `endTime` | Yes | string | Millisecond timestamp | `current timestamp` |
| `size` | Yes | string | Count | `10` |

#### `GET /api/longshort/realtimeAll` — Exchange Real-Time Long/Short Ratio

| Param | Required | Type | Description | Example |
|-------|----------|------|-------------|---------|
| `baseCoin` | Yes | string | Coin | `BTC` |
| `interval` | Yes | string | `5m/15m/30m/1h/2h/4h/6h/8h/12h/1d` | `1h` |

#### `GET /api/longshort/person` — Long/Short Account Ratio
**Supported: Binance / OKX / Bybit**

| Param | Required | Type | Description | Example |
|-------|----------|------|-------------|---------|
| `exchange` | Yes | string | Exchange | `Binance` |
| `symbol` | Yes | string | Trading pair | `BTCUSDT` |
| `interval` | Yes | string | Period | `1h` |
| `endTime` | Yes | string | Millisecond timestamp | `current timestamp` |
| `size` | Yes | string | Count, max 500 | `10` |

#### `GET /api/longshort/position` — Top Trader Long/Short (Position)
**Supported: Binance / OKX / Huobi** — Same params as `person`.

#### `GET /api/longshort/account` — Top Trader Long/Short (Account)
**Supported: Binance / OKX / Huobi** — Same params as `person`.

#### `GET /api/longshort/kline` — Long/Short Ratio K-Line
**Supported: Binance / OKX / Huobi**

| Param | Required | Type | Description | Example |
|-------|----------|------|-------------|---------|
| `exchange` | Yes | string | Exchange | `Binance` |
| `symbol` | Yes | string | Trading pair | `BTCUSDT` |
| `interval` | Yes | string | Period | `1h` |
| `endTime` | Yes | string | Millisecond timestamp | `current timestamp` |
| `size` | Yes | string | Count, max 500 | `10` |
| `type` | Yes | string | `longShortPerson` / `longShortPosition` / `longShortAccount` | `longShortPerson` |

```bash
curl -H "apikey: $APIKEY" \
  "https://open-api.coinank.com/api/longshort/realtimeAll?baseCoin=BTC&interval=1h"

curl -H "apikey: $APIKEY" \
  "https://open-api.coinank.com/api/longshort/person?exchange=Binance&symbol=BTCUSDT&interval=1h&endTime=$NOW&size=10"

curl -H "apikey: $APIKEY" \
  "https://open-api.coinank.com/api/longshort/kline?exchange=Binance&symbol=BTCUSDT&interval=1h&endTime=$NOW&size=10&type=longShortPerson"
```

</details>

---

<details>
<summary><strong>8. Market Order Stats</strong> — 8 endpoints · VIP3</summary>

<br />

> Split into **per-symbol** and **aggregated (cross-exchange)** groups.

#### Per-Symbol Series (requires exchange + symbol)

| Endpoint | Description |
|----------|-------------|
| `GET /api/marketOrder/getCvd` | CVD (Cumulative Volume Delta) |
| `GET /api/marketOrder/getBuySellCount` | Taker buy/sell count |
| `GET /api/marketOrder/getBuySellValue` | Taker buy/sell value (USD) |
| `GET /api/marketOrder/getBuySellVolume` | Taker buy/sell volume (coin) |

**Common Parameters:**

| Param | Required | Type | Description | Example |
|-------|----------|------|-------------|---------|
| `exchange` | Yes | string | Exchange (Binance / OKX / Bybit / Bitget) | `Binance` |
| `symbol` | Yes | string | Trading pair | `BTCUSDT` |
| `interval` | Yes | string | Period | `1h` |
| `endTime` | Yes | string | Millisecond timestamp | `current timestamp` |
| `size` | Yes | integer | Count, max 500 | `10` |
| `productType` | Yes | string | `SWAP` / `SPOT` | `SWAP` |

#### Aggregated Series (by baseCoin across exchanges)

| Endpoint | Description |
|----------|-------------|
| `GET /api/marketOrder/getAggCvd` | Aggregated CVD |
| `GET /api/marketOrder/getAggBuySellCount` | Aggregated buy/sell count |
| `GET /api/marketOrder/getAggBuySellValue` | Aggregated buy/sell value |
| `GET /api/marketOrder/getAggBuySellVolume` | Aggregated buy/sell volume |

**Common Parameters:**

| Param | Required | Type | Description | Example |
|-------|----------|------|-------------|---------|
| `baseCoin` | Yes | string | Coin | `BTC` |
| `interval` | Yes | string | Period | `1h` |
| `endTime` | Yes | string | Millisecond timestamp | `current timestamp` |
| `size` | Yes | integer | Count, max 500 | `10` |
| `productType` | Yes | string | `SWAP` / `SPOT` | `SWAP` |
| `exchanges` | Yes | string | **Pass empty string** to aggregate all | `(empty)` |

```bash
curl -H "apikey: $APIKEY" \
  "https://open-api.coinank.com/api/marketOrder/getCvd?exchange=Binance&symbol=BTCUSDT&interval=1h&endTime=$NOW&size=10&productType=SWAP"

# Note: exchanges param is required, pass empty string to aggregate all
curl -H "apikey: $APIKEY" \
  "https://open-api.coinank.com/api/marketOrder/getAggCvd?baseCoin=BTC&interval=1h&endTime=$NOW&size=10&productType=SWAP&exchanges="
```

</details>

---

<details>
<summary><strong>9. News & Flash</strong> — 2 endpoints · VIP2</summary>

<br />

#### `GET /api/news/getNewsList` — News/Flash List

| Param | Required | Type | Description | Example |
|-------|----------|------|-------------|---------|
| `type` | Yes | string | `1` flash / `2` news | `1` |
| `lang` | Yes | string | `zh` Chinese / `en` English | `en` |
| `page` | Yes | string | Page number | `1` |
| `pageSize` | Yes | string | Page size | `10` |
| `isPopular` | Yes | string | Featured: `true` / `false` | `false` |
| `search` | Yes | string | Search keyword, empty string for none | `(empty)` |

#### `GET /api/news/getNewsDetail` — News Detail

| Param | Required | Type | Description | Example |
|-------|----------|------|-------------|---------|
| `id` | Yes | string | News ID (from list endpoint) | `69a2f40912d08f6a781aedd0` |

```bash
curl -H "apikey: $APIKEY" \
  "https://open-api.coinank.com/api/news/getNewsList?type=1&lang=en&page=1&pageSize=10&isPopular=false&search="

curl -H "apikey: $APIKEY" \
  "https://open-api.coinank.com/api/news/getNewsDetail?id=69a2f40912d08f6a781aedd0"
```

</details>

---

<details>
<summary><strong>10. Indicators</strong> — 10 endpoints · VIP1</summary>

<br />

> The following indicators require no parameters.

| Endpoint | Description |
|----------|-------------|
| `GET /api/indicator/getBtcMultiplier` | 2-Year MA Multiplier |
| `GET /api/indicator/getCnnEntity` | Fear & Greed Index |
| `GET /api/indicator/getAhr999` | AHR999 Accumulation Index |
| `GET /api/indicator/getPuellMultiple` | Puell Multiple |
| `GET /api/indicator/getBtcPi` | Pi Cycle Top Indicator |
| `GET /api/indicator/getMovingAvgHeatmap` | 200-Week MA Heatmap |
| `GET /api/indicator/getAltcoinSeason` | Altcoin Season Index |

#### `GET /api/indicator/getMarketCapRank` — Market Cap Dominance Ranking

| Param | Required | Type | Description | Example |
|-------|----------|------|-------------|---------|
| `symbol` | Yes | string | Coin | `BTC` |

#### `GET /api/indicator/getGrayscaleOpenInterest` — Grayscale Holdings

| Param | Required | Type | Description | Example |
|-------|----------|------|-------------|---------|
| `symbol` | Yes | string | Coin | `BTC` |

#### `GET /api/indicator/index/charts` — Rainbow Chart & Composite Indicators

| Param | Required | Type | Description | Example |
|-------|----------|------|-------------|---------|
| `type` | Yes | string | Chart type | `bitcoin-rainbow-v2` |

```bash
curl -H "apikey: $APIKEY" "https://open-api.coinank.com/api/indicator/getCnnEntity"
curl -H "apikey: $APIKEY" "https://open-api.coinank.com/api/indicator/getMarketCapRank?symbol=BTC"
curl -H "apikey: $APIKEY" "https://open-api.coinank.com/api/indicator/index/charts?type=bitcoin-rainbow-v2"
```

</details>

---

<details>
<summary><strong>11. Open Interest</strong> — 7 endpoints · VIP1</summary>

<br />

#### `GET /api/openInterest/all` — Real-Time OI (All Exchanges)

| Param | Required | Type | Description | Example |
|-------|----------|------|-------------|---------|
| `baseCoin` | Yes | string | Coin | `BTC` |

#### `GET /api/openInterest/v2/chart` — Aggregated OI History

| Param | Required | Type | Description | Example |
|-------|----------|------|-------------|---------|
| `baseCoin` | Yes | string | Coin | `BTC` |
| `exchange` | Yes | string | Exchange, empty string for all | `(empty)` |
| `interval` | Yes | string | Period | `1h` |
| `size` | Yes | string | Count, max 500 | `10` |
| `type` | Yes | string | `USD` or coin name (e.g. `BTC`) | `USD` |
| `endTime` | Yes | number | Millisecond timestamp | `current timestamp` |

#### `GET /api/openInterest/symbol/Chart` — Symbol OI History

| Param | Required | Type | Description | Example |
|-------|----------|------|-------------|---------|
| `exchange` | Yes | string | Exchange | `Binance` |
| `symbol` | Yes | string | Trading pair | `BTCUSDT` |
| `interval` | Yes | string | Period | `1h` |
| `endTime` | Yes | number | Millisecond timestamp | `current timestamp` |
| `size` | Yes | string | Count, max 500 | `10` |
| `type` | Yes | string | `USD` / coin name | `USD` |

#### `GET /api/openInterest/kline` — Symbol OI K-Line

| Param | Required | Type | Description | Example |
|-------|----------|------|-------------|---------|
| `exchange` | Yes | string | Exchange | `Binance` |
| `symbol` | Yes | string | Trading pair | `BTCUSDT` |
| `interval` | Yes | string | Period | `1h` |
| `endTime` | Yes | number | Millisecond timestamp | `current timestamp` |
| `size` | Yes | integer | Count | `10` |

#### `GET /api/openInterest/aggKline` — Aggregated OI K-Line

| Param | Required | Type | Description | Example |
|-------|----------|------|-------------|---------|
| `baseCoin` | Yes | string | Coin | `BTC` |
| `interval` | Yes | string | Period | `1h` |
| `endTime` | Yes | number | Millisecond timestamp | `current timestamp` |
| `size` | Yes | integer | Count | `10` |

#### `GET /api/tickers/topOIByEx` — Real-Time OI by Exchange

| Param | Required | Type | Description | Example |
|-------|----------|------|-------------|---------|
| `baseCoin` | Yes | string | Coin | `BTC` |

#### `GET /api/instruments/oiVsMc` — Historical OI/Market Cap Ratio
**Tier: VIP2**

| Param | Required | Type | Description | Example |
|-------|----------|------|-------------|---------|
| `baseCoin` | Yes | string | Coin | `BTC` |
| `endTime` | Yes | string | Millisecond timestamp | `current timestamp` |
| `size` | Yes | string | Count, max 500 | `100` |
| `interval` | Yes | string | Period | `1h` |

```bash
curl -H "apikey: $APIKEY" \
  "https://open-api.coinank.com/api/openInterest/all?baseCoin=BTC"

curl -H "apikey: $APIKEY" \
  "https://open-api.coinank.com/api/openInterest/aggKline?baseCoin=BTC&interval=1h&endTime=$NOW&size=10"

curl -H "apikey: $APIKEY" \
  "https://open-api.coinank.com/api/instruments/oiVsMc?baseCoin=BTC&endTime=$NOW&size=100&interval=1h"
```

</details>

---

<details>
<summary><strong>12. Trending Rankings</strong> — 8 endpoints · VIP2</summary>

<br />

#### `GET /api/instruments/visualScreener` — Visual Screener

| Param | Required | Type | Description | Example |
|-------|----------|------|-------------|---------|
| `interval` | Yes | string | `15m` / `1h` / `4h` / `24h` | `15m` |

#### `GET /api/instruments/oiVsMarketCap` — OI/Market Cap Ranking

| Param | Required | Type | Description | Example |
|-------|----------|------|-------------|---------|
| `page` | Yes | integer | Page number | `1` |
| `size` | Yes | integer | Page size | `10` |
| `sortBy` | Yes | string | Sort field | `openInterest` |
| `sortType` | Yes | string | `desc` / `asc` | `desc` |

#### `GET /api/instruments/longShortRank` — Long/Short Ratio Ranking

| Param | Required | Type | Description | Example |
|-------|----------|------|-------------|---------|
| `sortBy` | Yes | string | Sort field | `longRatio` |
| `sortType` | Yes | string | `desc` / `asc` | `desc` |
| `size` | Yes | integer | Page size | `10` |
| `page` | Yes | integer | Page number | `1` |

#### `GET /api/instruments/oiRank` — Open Interest Ranking
Same params as `longShortRank`, `sortBy` example: `openInterest`.

#### `GET /api/trades/count` — Trade Count Ranking

| Param | Required | Type | Description | Example |
|-------|----------|------|-------------|---------|
| `productType` | Yes | string | `SWAP` / `SPOT` | `SWAP` |
| `sortBy` | Yes | string | e.g. `h1Count` (1h), `d1Count` (1d) | `h1Count` |
| `sortType` | Yes | string | `desc` / `asc` | `desc` |

#### `GET /api/instruments/liquidationRank` — Liquidation Ranking

| Param | Required | Type | Description | Example |
|-------|----------|------|-------------|---------|
| `sortBy` | Yes | string | e.g. `liquidationH24` | `liquidationH24` |
| `sortType` | Yes | string | `desc` / `asc` | `desc` |
| `page` | Yes | integer | Page number | `1` |
| `size` | Yes | integer | Page size | `10` |

#### `GET /api/instruments/priceRank` — Price Change Ranking

| Param | Required | Type | Description | Example |
|-------|----------|------|-------------|---------|
| `sortBy` | Yes | string | e.g. `priceChangeH24` | `priceChangeH24` |
| `sortType` | Yes | string | `desc` / `asc` | `desc` |

#### `GET /api/instruments/volumeRank` — Volume Change Ranking

| Param | Required | Type | Description | Example |
|-------|----------|------|-------------|---------|
| `sortBy` | Yes | string | e.g. `h24Volume` | `h24Volume` |
| `sortType` | Yes | string | `desc` / `asc` | `desc` |

```bash
curl -H "apikey: $APIKEY" \
  "https://open-api.coinank.com/api/instruments/visualScreener?interval=15m"

curl -H "apikey: $APIKEY" \
  "https://open-api.coinank.com/api/trades/count?productType=SWAP&sortBy=h1Count&sortType=desc"

curl -H "apikey: $APIKEY" \
  "https://open-api.coinank.com/api/instruments/priceRank?sortBy=priceChangeH24&sortType=desc"
```

</details>

---

<details>
<summary><strong>13. Liquidation</strong> — 8 endpoints · VIP1</summary>

<br />

#### `GET /api/liquidation/allExchange/intervals` — Real-Time Liquidation Stats

| Param | Required | Type | Description | Example |
|-------|----------|------|-------------|---------|
| `baseCoin` | Yes | string | Coin | `BTC` |

#### `GET /api/liquidation/aggregated-history` — Aggregated Liquidation History

| Param | Required | Type | Description | Example |
|-------|----------|------|-------------|---------|
| `baseCoin` | Yes | string | Coin | `BTC` |
| `interval` | Yes | string | Period | `1h` |
| `endTime` | Yes | number | Millisecond timestamp | `current timestamp` |
| `size` | Yes | integer | Count | `10` |

#### `GET /api/liquidation/history` — Symbol Liquidation History

| Param | Required | Type | Description | Example |
|-------|----------|------|-------------|---------|
| `exchange` | Yes | string | Exchange | `Binance` |
| `symbol` | Yes | string | Trading pair | `BTCUSDT` |
| `interval` | Yes | string | Period | `1h` |
| `endTime` | Yes | number | Millisecond timestamp | `current timestamp` |
| `size` | Yes | integer | Count | `10` |

#### `GET /api/liquidation/orders` — Liquidation Orders
**Tier: VIP3**

| Param | Required | Type | Description | Example |
|-------|----------|------|-------------|---------|
| `baseCoin` | Yes | string | Coin | `BTC` |
| `exchange` | Yes | string | Exchange | `Binance` |
| `side` | Yes | string | `long` / `short` | `long` |
| `amount` | Yes | number | Min liquidation amount (USD) | `100` |
| `endTime` | Yes | number | Millisecond timestamp | `current timestamp` |

#### `GET /api/liqMap/getLiqMap` — Liquidation Map
**Tier: VIP4**

| Param | Required | Type | Description | Example |
|-------|----------|------|-------------|---------|
| `symbol` | Yes | string | Trading pair | `BTCUSDT` |
| `exchange` | Yes | string | Exchange | `Binance` |
| `interval` | Yes | string | Period | `1d` |

#### `GET /api/liqMap/getAggLiqMap` — Aggregated Liquidation Map
**Tier: VIP4**

| Param | Required | Type | Description | Example |
|-------|----------|------|-------------|---------|
| `baseCoin` | Yes | string | Coin | `BTC` |
| `interval` | Yes | string | Period | `1d` |

#### `GET /api/liqMap/getLiqHeatMap` — Liquidation Heatmap
**Tier: VIP4**

| Param | Required | Type | Description | Example |
|-------|----------|------|-------------|---------|
| `exchange` | Yes | string | Exchange | `Binance` |
| `symbol` | Yes | string | Trading pair | `BTCUSDT` |
| `interval` | Yes | string | Period | `1d` |

#### `GET /api/liqMap/getLiqHeatMapSymbol` — Supported Heatmap Symbols
**Tier: VIP1 | No parameters required**

```bash
curl -H "apikey: $APIKEY" \
  "https://open-api.coinank.com/api/liquidation/allExchange/intervals?baseCoin=BTC"

curl -H "apikey: $APIKEY" \
  "https://open-api.coinank.com/api/liquidation/orders?baseCoin=BTC&exchange=Binance&side=long&amount=100&endTime=$NOW"

curl -H "apikey: $APIKEY" \
  "https://open-api.coinank.com/api/liqMap/getLiqHeatMapSymbol"
```

</details>

---

<details>
<summary><strong>14. Order Book</strong> — 3 endpoints · VIP3</summary>

<br />

#### `GET /api/orderBook/v2/bySymbol` — Order Book Depth History (by Symbol)

| Param | Required | Type | Description | Example |
|-------|----------|------|-------------|---------|
| `symbol` | Yes | string | Trading pair | `BTCUSDT` |
| `exchange` | Yes | string | Exchange | `Binance` |
| `rate` | Yes | number | Price precision ratio | `0.01` |
| `productType` | Yes | string | `SWAP` / `SPOT` | `SWAP` |
| `interval` | Yes | string | Period | `1h` |
| `endTime` | Yes | number | Millisecond timestamp | `current timestamp` |
| `size` | Yes | integer | Count, max 500 | `10` |

#### `GET /api/orderBook/v2/byExchange` — Order Book Depth History (by Exchange)

| Param | Required | Type | Description | Example |
|-------|----------|------|-------------|---------|
| `baseCoin` | Yes | string | Coin | `BTC` |
| `productType` | Yes | string | `SWAP` / `SPOT` | `SWAP` |
| `interval` | Yes | string | Period | `1h` |
| `endTime` | Yes | number | Millisecond timestamp | `current timestamp` |
| `size` | Yes | integer | Count, max 500 | `10` |
| `exchanges` | Yes | string | Exchange name | `Binance` |
| `type` | Yes | string | Price precision ratio | `0.01` |

#### `GET /api/orderBook/getHeatMap` — Order Book Liquidity Heatmap
**Tier: VIP4**

> The `endTime` parameter is validated by the CDN cache layer and must be a current millisecond timestamp.

| Param | Required | Type | Description | Example |
|-------|----------|------|-------------|---------|
| `exchange` | Yes | string | Exchange (currently Binance only) | `Binance` |
| `symbol` | Yes | string | Trading pair | `BTCUSDT` |
| `interval` | Yes | string | `1m` / `3m` / `5m` | `1m` |
| `endTime` | Yes | string | Millisecond timestamp (**must be current**, expired times return 401) | `current timestamp` |
| `size` | Yes | string | Count, max 500 | `10` |

```bash
curl -H "apikey: $APIKEY" \
  "https://open-api.coinank.com/api/orderBook/v2/bySymbol?symbol=BTCUSDT&exchange=Binance&rate=0.01&productType=SWAP&interval=1h&endTime=$NOW&size=10"

curl -H "apikey: $APIKEY" \
  "https://open-api.coinank.com/api/orderBook/getHeatMap?exchange=Binance&symbol=BTCUSDT&interval=1m&endTime=$NOW&size=10"
```

</details>

---

<details>
<summary><strong>15. Fund Flow</strong> — 2 endpoints · VIP3</summary>

<br />

#### `GET /api/fund/fundReal` — Real-Time Fund Flow

| Param | Required | Type | Description | Example |
|-------|----------|------|-------------|---------|
| `productType` | Yes | string | `SWAP` / `SPOT` | `SWAP` |
| `page` | Yes | integer | Page number | `1` |
| `size` | Yes | integer | Page size | `10` |
| `sortBy` | Yes | string | Sort field, e.g. `h1net` (1h net inflow) | `h1net` |
| `sortType` | Yes | string | `desc` / `asc` | `desc` |
| `baseCoin` | Yes | string | Coin (empty string for all) | `BTC` |

#### `GET /api/fund/getFundHisList` — Historical Fund Flow

| Param | Required | Type | Description | Example |
|-------|----------|------|-------------|---------|
| `baseCoin` | Yes | string | Coin | `BTC` |
| `endTime` | Yes | number | Millisecond timestamp | `current timestamp` |
| `productType` | Yes | string | `SWAP` / `SPOT` | `SWAP` |
| `size` | Yes | integer | Count | `10` |
| `interval` | Yes | string | Period | `1h` |

```bash
curl -H "apikey: $APIKEY" \
  "https://open-api.coinank.com/api/fund/fundReal?productType=SWAP&page=1&size=10&sortBy=h1net&sortType=desc&baseCoin=BTC"

curl -H "apikey: $APIKEY" \
  "https://open-api.coinank.com/api/fund/getFundHisList?baseCoin=BTC&endTime=$NOW&productType=SWAP&size=10&interval=1h"
```

</details>

---

<details>
<summary><strong>16. Order Flow</strong> — 1 endpoint · VIP3</summary>

<br />

#### `GET /api/orderFlow/lists` — Order Flow Data

| Param | Required | Type | Description | Example |
|-------|----------|------|-------------|---------|
| `exchange` | Yes | string | Exchange | `Binance` |
| `symbol` | Yes | string | Trading pair | `BTCUSDT` |
| `interval` | Yes | string | Period | `1h` |
| `endTime` | Yes | number | Millisecond timestamp | `current timestamp` |
| `size` | Yes | integer | Count | `10` |
| `productType` | Yes | string | `SWAP` / `SPOT` | `SWAP` |
| `tickCount` | Yes | integer | Tick count | `1` |

```bash
curl -H "apikey: $APIKEY" \
  "https://open-api.coinank.com/api/orderFlow/lists?exchange=Binance&symbol=BTCUSDT&interval=1h&endTime=$NOW&size=10&productType=SWAP&tickCount=1"
```

</details>

---

<details>
<summary><strong>17. Funding Rates</strong> — 7 endpoints · VIP1</summary>

<br />

#### `GET /api/fundingRate/hist` — Historical Funding Rates (Cross-Exchange)

| Param | Required | Type | Description | Example |
|-------|----------|------|-------------|---------|
| `baseCoin` | Yes | string | Coin | `BTC` |
| `exchangeType` | Yes | string | Quote type: `USDT` / `USD` (coin-margined) | `USDT` |
| `endTime` | Yes | number | Millisecond timestamp | `current timestamp` |
| `size` | Yes | integer | Count | `10` |

#### `GET /api/fundingRate/current` — Real-Time Funding Rate Ranking

| Param | Required | Type | Description | Example |
|-------|----------|------|-------------|---------|
| `type` | Yes | string | `current` / `day` / `week` / `month` / `year` | `current` |

#### `GET /api/fundingRate/accumulated` — Accumulated Funding Rate

| Param | Required | Type | Description | Example |
|-------|----------|------|-------------|---------|
| `type` | Yes | string | `day` / `week` / `month` / `year` | `day` |

#### `GET /api/fundingRate/indicator` — Symbol Funding Rate History

| Param | Required | Type | Description | Example |
|-------|----------|------|-------------|---------|
| `exchange` | Yes | string | Exchange (Binance / OKX / Bybit / Huobi / Gate / Bitget) | `Binance` |
| `symbol` | Yes | string | Trading pair | `BTCUSDT` |
| `interval` | Yes | string | Period | `1h` |
| `endTime` | Yes | string | Millisecond timestamp | `current timestamp` |
| `size` | Yes | string | Count, max 500 | `10` |

#### `GET /api/fundingRate/kline` — Funding Rate K-Line
Same params as `fundingRate/indicator`.

#### `GET /api/fundingRate/getWeiFr` — Weighted Funding Rate

| Param | Required | Type | Description | Example |
|-------|----------|------|-------------|---------|
| `baseCoin` | Yes | string | Coin | `BTC` |
| `interval` | Yes | string | Period | `1h` |
| `endTime` | Yes | string | Millisecond timestamp | `current timestamp` |
| `size` | Yes | string | Count, max 500 | `10` |

#### `GET /api/fundingRate/frHeatmap` — Funding Rate Heatmap

| Param | Required | Type | Description | Example |
|-------|----------|------|-------------|---------|
| `type` | Yes | string | `openInterest` by OI / `marketCap` by market cap | `marketCap` |
| `interval` | Yes | string | `1D` / `1W` / `1M` / `6M` | `1M` |

```bash
curl -H "apikey: $APIKEY" \
  "https://open-api.coinank.com/api/fundingRate/current?type=current"

curl -H "apikey: $APIKEY" \
  "https://open-api.coinank.com/api/fundingRate/accumulated?type=day"

curl -H "apikey: $APIKEY" \
  "https://open-api.coinank.com/api/fundingRate/frHeatmap?type=marketCap&interval=1M"

curl -H "apikey: $APIKEY" \
  "https://open-api.coinank.com/api/fundingRate/getWeiFr?baseCoin=BTC&interval=1h&endTime=$NOW&size=10"
```

</details>

---

<details>
<summary><strong>18. RSI Screener</strong> — 1 endpoint · VIP2</summary>

<br />

#### `GET /api/rsiMap/list` — RSI Indicator Screener

| Param | Required | Type | Description | Example |
|-------|----------|------|-------------|---------|
| `interval` | Yes | string | Period (note uppercase): `1H` / `4H` / `1D` | `1H` |
| `exchange` | Yes | string | Exchange | `Binance` |

```bash
curl -H "apikey: $APIKEY" \
  "https://open-api.coinank.com/api/rsiMap/list?interval=1H&exchange=Binance"
```

</details>

<div align="center">
<br />
<img src="https://raw.githubusercontent.com/andreasbm/readme/master/assets/lines/rainbow.png" alt="-----" />

## Enum Reference

</div>

<details>
<summary><strong>interval (K-Line / Historical Data Periods)</strong></summary>

<br />

| Value | Description |
|-------|-------------|
| `1m` | 1 minute |
| `3m` | 3 minutes |
| `5m` | 5 minutes |
| `15m` | 15 minutes |
| `30m` | 30 minutes |
| `1h` | 1 hour |
| `2h` | 2 hours |
| `4h` | 4 hours |
| `6h` | 6 hours |
| `8h` | 8 hours |
| `12h` | 12 hours |
| `1d` | 1 day |

> RSI Screener uses uppercase: `1H`, `4H`, `1D`
> Funding Rate Heatmap uses: `1D`, `1W`, `1M`, `6M`

</details>

<details>
<summary><strong>exchange (Major Exchanges)</strong></summary>

<br />

| Value | Exchange |
|-------|----------|
| `Binance` | Binance |
| `OKX` | OKX |
| `Bybit` | Bybit |
| `Bitget` | Bitget |
| `Gate` | Gate.io |
| `Huobi` | HTX (Huobi) |
| `Bitmex` | BitMEX |
| `dYdX` | dYdX |
| `Bitfinex` | Bitfinex |
| `CME` | CME Group |
| `Kraken` | Kraken |
| `Deribit` | Deribit |

</details>

<details>
<summary><strong>productType (Product Types)</strong></summary>

<br />

| Value | Description |
|-------|-------------|
| `SWAP` | Perpetual Futures |
| `SPOT` | Spot |
| `FUTURES` | Delivery Futures |

</details>

<details>
<summary><strong>sortBy Common Fields</strong></summary>

<br />

| Endpoint Type | Common sortBy Values |
|---------------|---------------------|
| OI Ranking | `openInterest` |
| Liquidation Ranking | `liquidationH24`, `liquidationH12`, `liquidationH8`, `liquidationH4`, `liquidationH1` |
| Price Ranking | `priceChangeH24`, `priceChangeH1`, `priceChangeM5` |
| Volume Ranking | `h24Volume`, `h1Volume` |
| Trade Count Ranking | `h1Count`, `d1Count`, `h4Count` |
| Fund Flow | `h1net`, `h4net`, `h8net`, `h24net` |
| Whale Positions | `positionValue`, `unrealizedPnl` |

</details>

<div align="center">
<br />
<img src="https://raw.githubusercontent.com/andreasbm/readme/master/assets/lines/rainbow.png" alt="-----" />

<br />

## License

[MIT License](./LICENSE) — CoinAnk

<br />

```
Built for AI-powered crypto derivatives intelligence.
```

<br />

<sub>Made by <a href="https://github.com/coinank">CoinAnk</a></sub>

</div>
