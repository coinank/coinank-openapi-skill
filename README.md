# CoinAnk OpenAPI Skill

> AI Agent 可读的 CoinAnk 开放接口调用技能。覆盖加密货币市场的 K 线、ETF、持仓、多空比、资金费率、爆仓、订单流等 18 大类、59 个接口，全部经过实测验证可用。

---

## 目录

- [快速开始](#快速开始)
- [认证与请求规范](#认证与请求规范)
- [⚠️ 关键注意事项](#️-关键注意事项)
- [接口总览](#接口总览)
- [接口详情](#接口详情)
  - [1. K线](#1-k线)
  - [2. ETF](#2-etf)
  - [3. HyperLiquid 鲸鱼](#3-hyperliquid-鲸鱼)
  - [4. 净多头和净空头](#4-净多头和净空头)
  - [5. 大额订单](#5-大额订单)
  - [6. 币种和交易对](#6-币种和交易对)
  - [7. 多空比](#7-多空比)
  - [8. 市价单统计指标](#8-市价单统计指标)
  - [9. 新闻快讯](#9-新闻快讯)
  - [10. 指标数据](#10-指标数据)
  - [11. 未平仓合约](#11-未平仓合约)
  - [12. 热门排行](#12-热门排行)
  - [13. 爆仓数据](#13-爆仓数据)
  - [14. 订单本](#14-订单本)
  - [15. 资金流](#15-资金流)
  - [16. 订单流](#16-订单流)
  - [17. 资金费率](#17-资金费率)
  - [18. RSI 选币器](#18-rsi-选币器)
- [枚举值速查](#枚举值速查)

---

## 快速开始

```bash
# 1. 设置 API Key（在 header 中传入）
APIKEY="your_api_key_here"

# 2. 生成当前毫秒级时间戳（macOS / Linux 通用）
NOW=$(python3 -c "import time; print(int(time.time()*1000))")

# 3. 示例：查询 BTC K线
curl -s -H "apikey: $APIKEY" \
  "https://open-api.coinank.com/api/kline/lists?symbol=BTCUSDT&exchange=Binance&endTime=$NOW&size=10&interval=1h&productType=SWAP"
```

---

## 认证与请求规范

| 项目 | 说明 |
|------|------|
| **Base URL** | `https://open-api.coinank.com` |
| **认证方式** | HTTP Header：`apikey: <your_api_key>` |
| **请求方法** | 全部为 `GET` |
| **响应格式** | `application/json` |
| **成功标志** | `{"success": true, "code": "1", "data": ...}` |

### 标准响应结构

```json
{
  "success": true,
  "code": "1",
  "data": [ ... ]
}
```

### 错误码说明

| code | 含义 |
|------|------|
| `1` | 成功 |
| `-3` | API Key 无效或认证失败 |
| `-7` | 超出允许访问的时间范围（endTime 参数错误） |
| `0` | 系统错误（参数缺失或服务端异常） |

---

## ⚠️ 关键注意事项

### 1. 时间戳必须是毫秒级且为当前时间

所有 `endTime` 参数均为**毫秒级时间戳**，且必须接近当前时间。传入过期或格式错误的时间戳会返回 `code: -7`。

```bash
# ✅ 正确：使用 python3 生成（跨平台兼容）
NOW=$(python3 -c "import time; print(int(time.time()*1000))")

# ❌ 错误：macOS 的 date 命令不支持 %3N，会生成如 "17228693N" 的无效值
NOW=$(date +%s%3N)  # 不要用这个！
```

### 2. 套餐权限等级

接口分为 VIP1～VIP4 四个级别，级别越高可访问的接口越多。每个接口标注了所需最低套餐。

### 3. `exchanges` 参数传空字符串

`getAggCvd`、`getAggBuySellCount` 等聚合市价单接口中，`exchanges` 参数**必须传入**（传空字符串 `exchanges=` 表示聚合所有交易所）。

### 4. OpenAPI 文件中的时间戳仅为示例

`references/` 目录下 JSON 文件中的 `example` 时间戳均为历史示例，调用时应使用实时生成的时间戳。

---

## 接口总览

| # | 分类 | 接口数 | 最低套餐 |
|---|------|--------|---------|
| 1 | K线 | 1 | VIP1 |
| 2 | ETF | 5 | VIP1 |
| 3 | HyperLiquid 鲸鱼 | 2 | VIP2 |
| 4 | 净多头和净空头 | 1 | VIP3 |
| 5 | 大额订单 | 2 | VIP3 |
| 6 | 币种和交易对 | 4 | VIP1 |
| 7 | 多空比 | 6 | VIP1 |
| 8 | 市价单统计指标 | 8 | VIP3 |
| 9 | 新闻快讯 | 2 | VIP2 |
| 10 | 指标数据 | 10 | VIP1 |
| 11 | 未平仓合约 | 7 | VIP1 |
| 12 | 热门排行 | 8 | VIP2 |
| 13 | 爆仓数据 | 8 | VIP1 |
| 14 | 订单本 | 3 | VIP3 |
| 15 | 资金流 | 2 | VIP3 |
| 16 | 订单流 | 1 | VIP3 |
| 17 | 资金费率 | 7 | VIP1 |
| 18 | RSI 选币器 | 1 | VIP2 |
| **合计** | | **59** | |

---

## 接口详情

---

### 1. K线

#### `GET /api/kline/lists` — K线行情数据
**套餐：VIP1**

| 参数 | 必填 | 类型 | 说明 | 示例 |
|------|------|------|------|------|
| `symbol` | ✅ | string | 交易对 | `BTCUSDT` |
| `exchange` | ✅ | string | 交易所 | `Binance` |
| `endTime` | ✅ | number | 毫秒时间戳，返回此时间之前的数据 | `当前时间戳` |
| `size` | ✅ | integer | 数量，最大 500 | `10` |
| `interval` | ✅ | string | 周期，见枚举值 | `1h` |
| `productType` | ✅ | string | `SWAP` 合约 / `SPOT` 现货 | `SWAP` |

```bash
curl -H "apikey: $APIKEY" \
  "https://open-api.coinank.com/api/kline/lists?symbol=BTCUSDT&exchange=Binance&endTime=$NOW&size=10&interval=1h&productType=SWAP"
```

---

### 2. ETF

#### `GET /api/etf/getUsBtcEtf` — 美国 BTC ETF 列表
**套餐：VIP1 | 无需参数**

#### `GET /api/etf/getUsEthEtf` — 美国 ETH ETF 列表
**套餐：VIP1 | 无需参数**

#### `GET /api/etf/usBtcInflow` — 美国 BTC ETF 历史净流入
**套餐：VIP1 | 无需参数**

#### `GET /api/etf/usEthInflow` — 美国 ETH ETF 历史净流入
**套餐：VIP1 | 无需参数**

#### `GET /api/etf/hkEtfInflow` — 港股 ETF 历史净流入
**套餐：VIP1 | 无需参数**

```bash
curl -H "apikey: $APIKEY" "https://open-api.coinank.com/api/etf/getUsBtcEtf"
curl -H "apikey: $APIKEY" "https://open-api.coinank.com/api/etf/getUsEthEtf"
curl -H "apikey: $APIKEY" "https://open-api.coinank.com/api/etf/usBtcInflow"
curl -H "apikey: $APIKEY" "https://open-api.coinank.com/api/etf/usEthInflow"
curl -H "apikey: $APIKEY" "https://open-api.coinank.com/api/etf/hkEtfInflow"
```

---

### 3. HyperLiquid 鲸鱼

#### `GET /api/hyper/topPosition` — 鲸鱼持仓排行
**套餐：VIP2**

| 参数 | 必填 | 类型 | 说明 | 示例 |
|------|------|------|------|------|
| `sortBy` | ✅ | string | 排序字段 | `positionValue` |
| `sortType` | ✅ | string | `desc` 降序 / `asc` 升序 | `desc` |
| `page` | ✅ | integer | 页码 | `1` |
| `size` | ✅ | integer | 每页数量 | `10` |

#### `GET /api/hyper/topAction` — 鲸鱼最新动态
**套餐：VIP2 | 无需参数**

```bash
curl -H "apikey: $APIKEY" \
  "https://open-api.coinank.com/api/hyper/topPosition?sortBy=positionValue&sortType=desc&page=1&size=10"

curl -H "apikey: $APIKEY" "https://open-api.coinank.com/api/hyper/topAction"
```

---

### 4. 净多头和净空头

#### `GET /api/netPositions/getNetPositions` — 净多头/净空头历史
**套餐：VIP3**

| 参数 | 必填 | 类型 | 说明 | 示例 |
|------|------|------|------|------|
| `exchange` | ✅ | string | 交易所 | `Binance` |
| `symbol` | ✅ | string | 交易对 | `BTCUSDT` |
| `interval` | ✅ | string | 周期 | `1h` |
| `endTime` | ✅ | number | 毫秒时间戳 | `当前时间戳` |
| `size` | ✅ | integer | 数量，最大 500 | `10` |

```bash
curl -H "apikey: $APIKEY" \
  "https://open-api.coinank.com/api/netPositions/getNetPositions?exchange=Binance&symbol=BTCUSDT&interval=1h&endTime=$NOW&size=10"
```

---

### 5. 大额订单

#### `GET /api/trades/largeTrades` — 大额市价订单
**套餐：VIP3**

| 参数 | 必填 | 类型 | 说明 | 示例 |
|------|------|------|------|------|
| `symbol` | ✅ | string | 交易对 | `BTCUSDT` |
| `productType` | ✅ | string | `SWAP` 合约 / `SPOT` 现货 | `SWAP` |
| `amount` | ✅ | string | 最小金额（USD） | `10000000` |
| `endTime` | ✅ | string | 毫秒时间戳 | `当前时间戳` |
| `size` | ✅ | string | 数量，最大 500 | `10` |

```bash
curl -H "apikey: $APIKEY" \
  "https://open-api.coinank.com/api/trades/largeTrades?symbol=BTCUSDT&productType=SWAP&amount=10000000&endTime=$NOW&size=10"
```

#### `GET /api/bigOrder/queryOrderList` — 大额限价订单
**套餐：VIP3**

| 参数 | 必填 | 类型 | 说明 | 示例 |
|------|------|------|------|------|
| `symbol` | ✅ | string | 交易对 | `BTCUSDT` |
| `exchangeType` | ✅ | string | `SWAP` 永续 / `SPOT` 现货 / `FUTURES` 交割 | `SWAP` |
| `size` | ✅ | integer | 数量，最大 500 | `10` |
| `amount` | ✅ | number | 最低金额（USD） | `1000000` |
| `side` | ✅ | string | `ask` 卖 / `bid` 买 | `ask` |
| `exchange` | ✅ | string | 交易所（Binance / OKX / Coinbase） | `Binance` |
| `isHistory` | ✅ | string | `true` 历史 / `false` 实时 | `true` |
| `startTime` | ❌ | number | 截止时间戳（isHistory=true 时建议传当前时间戳） | `当前时间戳` |

```bash
curl -H "apikey: $APIKEY" \
  "https://open-api.coinank.com/api/bigOrder/queryOrderList?symbol=BTCUSDT&exchangeType=SWAP&size=10&amount=1000000&side=ask&exchange=Binance&isHistory=true&startTime=$NOW"
```

---

### 6. 币种和交易对

#### `GET /api/instruments/getLastPrice` — 实时价格
**套餐：VIP1**

| 参数 | 必填 | 类型 | 说明 | 示例 |
|------|------|------|------|------|
| `symbol` | ✅ | string | 交易对 | `BTCUSDT` |
| `exchange` | ✅ | string | 交易所 | `Binance` |
| `productType` | ✅ | string | `SWAP` / `SPOT` | `SWAP` |

#### `GET /api/instruments/getCoinMarketCap` — 币种市值信息
**套餐：VIP1**

| 参数 | 必填 | 类型 | 说明 | 示例 |
|------|------|------|------|------|
| `baseCoin` | ✅ | string | 币种 | `BTC` |

#### `GET /api/baseCoin/list` — 支持的币种列表
**套餐：VIP1**

| 参数 | 必填 | 类型 | 说明 | 示例 |
|------|------|------|------|------|
| `productType` | ✅ | string | `SWAP` / `SPOT` | `SWAP` |

#### `GET /api/baseCoin/symbols` — 支持的交易对列表
**套餐：VIP1**

| 参数 | 必填 | 类型 | 说明 | 示例 |
|------|------|------|------|------|
| `exchange` | ✅ | string | 交易所 | `Binance` |
| `productType` | ✅ | string | `SWAP` / `SPOT` | `SWAP` |

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

---

### 7. 多空比

#### `GET /api/longshort/buySell` — 全市场多空买卖比
**套餐：VIP3**

| 参数 | 必填 | 类型 | 说明 | 示例 |
|------|------|------|------|------|
| `baseCoin` | ✅ | string | 币种 | `BTC` |
| `interval` | ✅ | string | 周期 | `1h` |
| `endTime` | ✅ | string | 毫秒时间戳 | `当前时间戳` |
| `size` | ✅ | string | 数量 | `10` |

#### `GET /api/longshort/realtimeAll` — 交易所实时多空比
**套餐：VIP1**

| 参数 | 必填 | 类型 | 说明 | 示例 |
|------|------|------|------|------|
| `baseCoin` | ✅ | string | 币种 | `BTC` |
| `interval` | ✅ | string | 周期，可选 `5m/15m/30m/1h/2h/4h/6h/8h/12h/1d` | `1h` |

#### `GET /api/longshort/person` — 多空持仓人数比
**套餐：VIP1 | 支持交易所：Binance / OKX / Bybit**

| 参数 | 必填 | 类型 | 说明 | 示例 |
|------|------|------|------|------|
| `exchange` | ✅ | string | 交易所 | `Binance` |
| `symbol` | ✅ | string | 交易对 | `BTCUSDT` |
| `interval` | ✅ | string | 周期 | `1h` |
| `endTime` | ✅ | string | 毫秒时间戳 | `当前时间戳` |
| `size` | ✅ | string | 数量，最大 500 | `10` |

#### `GET /api/longshort/position` — 大户多空比（持仓量）
**套餐：VIP1 | 支持交易所：Binance / OKX / Huobi**

参数与 `/api/longshort/person` 相同。

#### `GET /api/longshort/account` — 大户多空比（账户数）
**套餐：VIP1 | 支持交易所：Binance / OKX / Huobi**

参数与 `/api/longshort/person` 相同。

#### `GET /api/longshort/kline` — 多空比 K 线
**套餐：VIP1 | 支持交易所：Binance / OKX / Huobi**

| 参数 | 必填 | 类型 | 说明 | 示例 |
|------|------|------|------|------|
| `exchange` | ✅ | string | 交易所 | `Binance` |
| `symbol` | ✅ | string | 交易对 | `BTCUSDT` |
| `interval` | ✅ | string | 周期 | `1h` |
| `endTime` | ✅ | string | 毫秒时间戳 | `当前时间戳` |
| `size` | ✅ | string | 数量，最大 500 | `10` |
| `type` | ✅ | string | `longShortPerson` 人数比 / `longShortPosition` 持仓比 / `longShortAccount` 账户比 | `longShortPerson` |

```bash
curl -H "apikey: $APIKEY" \
  "https://open-api.coinank.com/api/longshort/realtimeAll?baseCoin=BTC&interval=1h"

curl -H "apikey: $APIKEY" \
  "https://open-api.coinank.com/api/longshort/person?exchange=Binance&symbol=BTCUSDT&interval=1h&endTime=$NOW&size=10"

curl -H "apikey: $APIKEY" \
  "https://open-api.coinank.com/api/longshort/kline?exchange=Binance&symbol=BTCUSDT&interval=1h&endTime=$NOW&size=10&type=longShortPerson"
```

---

### 8. 市价单统计指标

> 以下 8 个接口均为 **VIP3**，分为**单交易对**和**聚合（跨交易所）**两组。

#### 单交易对系列（需指定 exchange + symbol）

| 接口 | 说明 |
|------|------|
| `GET /api/marketOrder/getCvd` | CVD（主动买卖量差） |
| `GET /api/marketOrder/getBuySellCount` | 主动买卖笔数 |
| `GET /api/marketOrder/getBuySellValue` | 主动买卖额（USD） |
| `GET /api/marketOrder/getBuySellVolume` | 主动买卖量（币本位） |

**公共参数：**

| 参数 | 必填 | 类型 | 说明 | 示例 |
|------|------|------|------|------|
| `exchange` | ✅ | string | 交易所（Binance / OKX / Bybit / Bitget） | `Binance` |
| `symbol` | ✅ | string | 交易对 | `BTCUSDT` |
| `interval` | ✅ | string | 周期 | `1h` |
| `endTime` | ✅ | string | 毫秒时间戳 | `当前时间戳` |
| `size` | ✅ | integer | 数量，最大 500 | `10` |
| `productType` | ✅ | string | `SWAP` / `SPOT` | `SWAP` |

```bash
curl -H "apikey: $APIKEY" \
  "https://open-api.coinank.com/api/marketOrder/getCvd?exchange=Binance&symbol=BTCUSDT&interval=1h&endTime=$NOW&size=10&productType=SWAP"
```

#### 聚合系列（按 baseCoin 跨交易所聚合）

| 接口 | 说明 |
|------|------|
| `GET /api/marketOrder/getAggCvd` | 聚合 CVD |
| `GET /api/marketOrder/getAggBuySellCount` | 聚合买卖笔数 |
| `GET /api/marketOrder/getAggBuySellValue` | 聚合买卖额 |
| `GET /api/marketOrder/getAggBuySellVolume` | 聚合买卖量 |

**公共参数：**

| 参数 | 必填 | 类型 | 说明 | 示例 |
|------|------|------|------|------|
| `baseCoin` | ✅ | string | 币种 | `BTC` |
| `interval` | ✅ | string | 周期 | `1h` |
| `endTime` | ✅ | string | 毫秒时间戳 | `当前时间戳` |
| `size` | ✅ | integer | 数量，最大 500 | `10` |
| `productType` | ✅ | string | `SWAP` / `SPOT` | `SWAP` |
| `exchanges` | ✅ | string | **传空字符串**表示聚合所有交易所 | `（空）` |

```bash
# 注意：exchanges 参数必须传入，传空字符串表示聚合全部
curl -H "apikey: $APIKEY" \
  "https://open-api.coinank.com/api/marketOrder/getAggCvd?baseCoin=BTC&interval=1h&endTime=$NOW&size=10&productType=SWAP&exchanges="
```

---

### 9. 新闻快讯

#### `GET /api/news/getNewsList` — 新闻/快讯列表
**套餐：VIP2**

| 参数 | 必填 | 类型 | 说明 | 示例 |
|------|------|------|------|------|
| `type` | ✅ | string | `1` 快讯 / `2` 新闻 | `1` |
| `lang` | ✅ | string | 语言：`zh` 中文 / `en` 英文 | `zh` |
| `page` | ✅ | string | 页码 | `1` |
| `pageSize` | ✅ | string | 每页数量 | `10` |
| `isPopular` | ✅ | string | 是否推荐：`true` / `false` | `false` |
| `search` | ✅ | string | 搜索关键词，无则传空字符串 | `（空）` |

#### `GET /api/news/getNewsDetail` — 新闻详情
**套餐：VIP2**

| 参数 | 必填 | 类型 | 说明 | 示例 |
|------|------|------|------|------|
| `id` | ✅ | string | 新闻 ID（从列表接口获取） | `69a2f40912d08f6a781aedd0` |

```bash
# 先获取列表，取 id
curl -H "apikey: $APIKEY" \
  "https://open-api.coinank.com/api/news/getNewsList?type=1&lang=zh&page=1&pageSize=10&isPopular=false&search="

# 再查详情
curl -H "apikey: $APIKEY" \
  "https://open-api.coinank.com/api/news/getNewsDetail?id=69a2f40912d08f6a781aedd0"
```

---

### 10. 指标数据

> 以下指标均为 **VIP1**，无需参数的直接请求即可。

| 接口 | 说明 | 参数 |
|------|------|------|
| `GET /api/indicator/getBtcMultiplier` | 两年 MA 乘数 | 无 |
| `GET /api/indicator/getCnnEntity` | 贪婪恐惧指数 | 无 |
| `GET /api/indicator/getAhr999` | ahr999 囤币指标 | 无 |
| `GET /api/indicator/getPuellMultiple` | 普尔系数（Puell Multiple） | 无 |
| `GET /api/indicator/getBtcPi` | Pi 循环顶部指标 | 无 |
| `GET /api/indicator/getMovingAvgHeatmap` | 200 周均线热力图 | 无 |
| `GET /api/indicator/getAltcoinSeason` | 山寨季指数 | 无 |

#### `GET /api/indicator/getMarketCapRank` — 市值占比排名

| 参数 | 必填 | 类型 | 说明 | 示例 |
|------|------|------|------|------|
| `symbol` | ✅ | string | 币种 | `BTC` |

#### `GET /api/indicator/getGrayscaleOpenInterest` — 灰度持仓数据

| 参数 | 必填 | 类型 | 说明 | 示例 |
|------|------|------|------|------|
| `symbol` | ✅ | string | 币种 | `BTC` |

#### `GET /api/indicator/index/charts` — 彩虹图等综合指标

| 参数 | 必填 | 类型 | 说明 | 示例 |
|------|------|------|------|------|
| `type` | ✅ | string | 图表类型 | `bitcoin-rainbow-v2` |

```bash
curl -H "apikey: $APIKEY" "https://open-api.coinank.com/api/indicator/getCnnEntity"
curl -H "apikey: $APIKEY" "https://open-api.coinank.com/api/indicator/getMarketCapRank?symbol=BTC"
curl -H "apikey: $APIKEY" "https://open-api.coinank.com/api/indicator/index/charts?type=bitcoin-rainbow-v2"
```

---

### 11. 未平仓合约

#### `GET /api/openInterest/all` — 实时持仓列表（全交易所）
**套餐：VIP1**

| 参数 | 必填 | 类型 | 说明 | 示例 |
|------|------|------|------|------|
| `baseCoin` | ✅ | string | 币种 | `BTC` |

#### `GET /api/openInterest/v2/chart` — 币种聚合持仓历史
**套餐：VIP1**

| 参数 | 必填 | 类型 | 说明 | 示例 |
|------|------|------|------|------|
| `baseCoin` | ✅ | string | 币种 | `BTC` |
| `exchange` | ✅ | string | 交易所，传空字符串查全部 | `（空）` |
| `interval` | ✅ | string | 周期 | `1h` |
| `size` | ✅ | string | 数量，最大 500 | `10` |
| `type` | ✅ | string | `USD` 美元计价 / 币种名（如 `BTC`）币本位 | `USD` |
| `endTime` | ✅ | number | 毫秒时间戳（可选，不传则返回最新） | `当前时间戳` |

#### `GET /api/openInterest/symbol/Chart` — 交易对持仓历史
**套餐：VIP1**

| 参数 | 必填 | 类型 | 说明 | 示例 |
|------|------|------|------|------|
| `exchange` | ✅ | string | 交易所 | `Binance` |
| `symbol` | ✅ | string | 交易对 | `BTCUSDT` |
| `interval` | ✅ | string | 周期 | `1h` |
| `endTime` | ✅ | number | 毫秒时间戳 | `当前时间戳` |
| `size` | ✅ | string | 数量，最大 500 | `10` |
| `type` | ✅ | string | `USD` / 币种名 | `USD` |

#### `GET /api/openInterest/kline` — 交易对持仓 K 线
**套餐：VIP1**

| 参数 | 必填 | 类型 | 说明 | 示例 |
|------|------|------|------|------|
| `exchange` | ✅ | string | 交易所 | `Binance` |
| `symbol` | ✅ | string | 交易对 | `BTCUSDT` |
| `interval` | ✅ | string | 周期 | `1h` |
| `endTime` | ✅ | number | 毫秒时间戳 | `当前时间戳` |
| `size` | ✅ | integer | 数量 | `10` |

#### `GET /api/openInterest/aggKline` — 聚合持仓 K 线
**套餐：VIP1**

| 参数 | 必填 | 类型 | 说明 | 示例 |
|------|------|------|------|------|
| `baseCoin` | ✅ | string | 币种 | `BTC` |
| `interval` | ✅ | string | 周期 | `1h` |
| `endTime` | ✅ | number | 毫秒时间戳 | `当前时间戳` |
| `size` | ✅ | integer | 数量 | `10` |

#### `GET /api/tickers/topOIByEx` — 实时持仓（按交易所统计）
**套餐：VIP1**

| 参数 | 必填 | 类型 | 说明 | 示例 |
|------|------|------|------|------|
| `baseCoin` | ✅ | string | 币种 | `BTC` |

#### `GET /api/instruments/oiVsMc` — 历史持仓市值比
**套餐：VIP2**

| 参数 | 必填 | 类型 | 说明 | 示例 |
|------|------|------|------|------|
| `baseCoin` | ✅ | string | 币种 | `BTC` |
| `endTime` | ✅ | string | 毫秒时间戳 | `当前时间戳` |
| `size` | ✅ | string | 数量，最大 500 | `100` |
| `interval` | ✅ | string | 周期 | `1h` |

```bash
curl -H "apikey: $APIKEY" \
  "https://open-api.coinank.com/api/openInterest/all?baseCoin=BTC"

curl -H "apikey: $APIKEY" \
  "https://open-api.coinank.com/api/openInterest/aggKline?baseCoin=BTC&interval=1h&endTime=$NOW&size=10"

curl -H "apikey: $APIKEY" \
  "https://open-api.coinank.com/api/instruments/oiVsMc?baseCoin=BTC&endTime=$NOW&size=100&interval=1h"
```

---

### 12. 热门排行

> 以下接口均为 **VIP2**。

#### `GET /api/instruments/visualScreener` — 视觉筛选器

| 参数 | 必填 | 类型 | 说明 | 示例 |
|------|------|------|------|------|
| `interval` | ✅ | string | `15m` / `1h` / `4h` / `24h` | `15m` |

#### `GET /api/instruments/oiVsMarketCap` — 持仓/市值排行

| 参数 | 必填 | 类型 | 说明 | 示例 |
|------|------|------|------|------|
| `page` | ✅ | integer | 页码 | `1` |
| `size` | ✅ | integer | 每页数量 | `10` |
| `sortBy` | ✅ | string | 排序字段 | `openInterest` |
| `sortType` | ✅ | string | `desc` / `asc` | `desc` |

#### `GET /api/instruments/longShortRank` — 多空持仓人数比排行

| 参数 | 必填 | 类型 | 说明 | 示例 |
|------|------|------|------|------|
| `sortBy` | ✅ | string | 排序字段 | `longRatio` |
| `sortType` | ✅ | string | `desc` / `asc` | `desc` |
| `size` | ✅ | integer | 每页数量 | `10` |
| `page` | ✅ | integer | 页码 | `1` |

#### `GET /api/instruments/oiRank` — 持仓量排行榜

参数同 `longShortRank`，`sortBy` 示例值：`openInterest`。

#### `GET /api/trades/count` — 交易笔数排行

| 参数 | 必填 | 类型 | 说明 | 示例 |
|------|------|------|------|------|
| `productType` | ✅ | string | `SWAP` / `SPOT` | `SWAP` |
| `sortBy` | ✅ | string | 排序字段，如 `h1Count`（1小时）、`d1Count`（1天） | `h1Count` |
| `sortType` | ✅ | string | `desc` / `asc` | `desc` |

#### `GET /api/instruments/liquidationRank` — 爆仓排行榜

| 参数 | 必填 | 类型 | 说明 | 示例 |
|------|------|------|------|------|
| `sortBy` | ✅ | string | 排序字段，如 `liquidationH24` | `liquidationH24` |
| `sortType` | ✅ | string | `desc` / `asc` | `desc` |
| `page` | ✅ | integer | 页码 | `1` |
| `size` | ✅ | integer | 每页数量 | `10` |

#### `GET /api/instruments/priceRank` — 价格变化排行

| 参数 | 必填 | 类型 | 说明 | 示例 |
|------|------|------|------|------|
| `sortBy` | ✅ | string | 如 `priceChangeH24`（24h涨跌幅） | `priceChangeH24` |
| `sortType` | ✅ | string | `desc` / `asc` | `desc` |

#### `GET /api/instruments/volumeRank` — 交易量变化排行

| 参数 | 必填 | 类型 | 说明 | 示例 |
|------|------|------|------|------|
| `sortBy` | ✅ | string | 如 `h24Volume`（24h交易量） | `h24Volume` |
| `sortType` | ✅ | string | `desc` / `asc` | `desc` |

```bash
curl -H "apikey: $APIKEY" \
  "https://open-api.coinank.com/api/instruments/visualScreener?interval=15m"

curl -H "apikey: $APIKEY" \
  "https://open-api.coinank.com/api/trades/count?productType=SWAP&sortBy=h1Count&sortType=desc"

curl -H "apikey: $APIKEY" \
  "https://open-api.coinank.com/api/instruments/priceRank?sortBy=priceChangeH24&sortType=desc"
```

---

### 13. 爆仓数据

#### `GET /api/liquidation/allExchange/intervals` — 各时间段实时爆仓统计
**套餐：VIP1**

| 参数 | 必填 | 类型 | 说明 | 示例 |
|------|------|------|------|------|
| `baseCoin` | ✅ | string | 币种 | `BTC` |

#### `GET /api/liquidation/aggregated-history` — 聚合爆仓历史
**套餐：VIP1**

| 参数 | 必填 | 类型 | 说明 | 示例 |
|------|------|------|------|------|
| `baseCoin` | ✅ | string | 币种 | `BTC` |
| `interval` | ✅ | string | 周期 | `1h` |
| `endTime` | ✅ | number | 毫秒时间戳 | `当前时间戳` |
| `size` | ✅ | integer | 数量 | `10` |

#### `GET /api/liquidation/history` — 交易对爆仓历史
**套餐：VIP1**

| 参数 | 必填 | 类型 | 说明 | 示例 |
|------|------|------|------|------|
| `exchange` | ✅ | string | 交易所 | `Binance` |
| `symbol` | ✅ | string | 交易对 | `BTCUSDT` |
| `interval` | ✅ | string | 周期 | `1h` |
| `endTime` | ✅ | number | 毫秒时间戳 | `当前时间戳` |
| `size` | ✅ | integer | 数量 | `10` |

#### `GET /api/liquidation/orders` — 爆仓订单列表
**套餐：VIP3**

| 参数 | 必填 | 类型 | 说明 | 示例 |
|------|------|------|------|------|
| `baseCoin` | ✅ | string | 币种 | `BTC` |
| `exchange` | ✅ | string | 交易所 | `Binance` |
| `side` | ✅ | string | `long` 多 / `short` 空 | `long` |
| `amount` | ✅ | number | 最低爆仓金额（USD） | `100` |
| `endTime` | ✅ | number | 毫秒时间戳 | `当前时间戳` |

#### `GET /api/liqMap/getLiqMap` — 清算地图
**套餐：VIP4**

| 参数 | 必填 | 类型 | 说明 | 示例 |
|------|------|------|------|------|
| `symbol` | ✅ | string | 交易对 | `BTCUSDT` |
| `exchange` | ✅ | string | 交易所 | `Binance` |
| `interval` | ✅ | string | 周期 | `1d` |

#### `GET /api/liqMap/getAggLiqMap` — 聚合清算地图
**套餐：VIP4**

| 参数 | 必填 | 类型 | 说明 | 示例 |
|------|------|------|------|------|
| `baseCoin` | ✅ | string | 币种 | `BTC` |
| `interval` | ✅ | string | 周期 | `1d` |

#### `GET /api/liqMap/getLiqHeatMap` — 清算热力图
**套餐：VIP4**

| 参数 | 必填 | 类型 | 说明 | 示例 |
|------|------|------|------|------|
| `exchange` | ✅ | string | 交易所 | `Binance` |
| `symbol` | ✅ | string | 交易对 | `BTCUSDT` |
| `interval` | ✅ | string | 周期 | `1d` |

#### `GET /api/liqMap/getLiqHeatMapSymbol` — 清算热图支持的交易对列表
**套餐：VIP1 | 无需参数**

```bash
curl -H "apikey: $APIKEY" \
  "https://open-api.coinank.com/api/liquidation/allExchange/intervals?baseCoin=BTC"

curl -H "apikey: $APIKEY" \
  "https://open-api.coinank.com/api/liquidation/orders?baseCoin=BTC&exchange=Binance&side=long&amount=100&endTime=$NOW"

curl -H "apikey: $APIKEY" \
  "https://open-api.coinank.com/api/liqMap/getLiqHeatMapSymbol"
```

---

### 14. 订单本

#### `GET /api/orderBook/v2/bySymbol` — 按交易对查询挂单深度历史
**套餐：VIP3**

| 参数 | 必填 | 类型 | 说明 | 示例 |
|------|------|------|------|------|
| `symbol` | ✅ | string | 交易对 | `BTCUSDT` |
| `exchange` | ✅ | string | 交易所 | `Binance` |
| `rate` | ✅ | number | 价格精度比例 | `0.01` |
| `productType` | ✅ | string | `SWAP` / `SPOT` | `SWAP` |
| `interval` | ✅ | string | 周期 | `1h` |
| `endTime` | ✅ | number | 毫秒时间戳 | `当前时间戳` |
| `size` | ✅ | integer | 数量，最大 500 | `10` |

#### `GET /api/orderBook/v2/byExchange` — 按交易所查询挂单深度历史
**套餐：VIP3**

| 参数 | 必填 | 类型 | 说明 | 示例 |
|------|------|------|------|------|
| `baseCoin` | ✅ | string | 币种 | `BTC` |
| `productType` | ✅ | string | `SWAP` / `SPOT` | `SWAP` |
| `interval` | ✅ | string | 周期 | `1h` |
| `endTime` | ✅ | number | 毫秒时间戳 | `当前时间戳` |
| `size` | ✅ | integer | 数量，最大 500 | `10` |
| `exchanges` | ✅ | string | 交易所名称 | `Binance` |
| `type` | ✅ | string | 价格精度比例 | `0.01` |

#### `GET /api/orderBook/getHeatMap` — 挂单流动性热力图
**套餐：VIP4**

> ⚠️ 此接口 `endTime` 参数会被 CDN 缓存层校验，必须传入当前毫秒时间戳。

| 参数 | 必填 | 类型 | 说明 | 示例 |
|------|------|------|------|------|
| `exchange` | ✅ | string | 交易所（目前仅支持 Binance） | `Binance` |
| `symbol` | ✅ | string | 交易对 | `BTCUSDT` |
| `interval` | ✅ | string | 周期：`1m` / `3m` / `5m` | `1m` |
| `endTime` | ✅ | string | 毫秒时间戳（**必须传当前时间**，过期时间会被 CDN 拦截返回 401） | `当前时间戳` |
| `size` | ✅ | string | 数量，最大 500 | `10` |

```bash
curl -H "apikey: $APIKEY" \
  "https://open-api.coinank.com/api/orderBook/v2/bySymbol?symbol=BTCUSDT&exchange=Binance&rate=0.01&productType=SWAP&interval=1h&endTime=$NOW&size=10"

curl -H "apikey: $APIKEY" \
  "https://open-api.coinank.com/api/orderBook/getHeatMap?exchange=Binance&symbol=BTCUSDT&interval=1m&endTime=$NOW&size=10"
```

---

### 15. 资金流

#### `GET /api/fund/fundReal` — 实时资金流
**套餐：VIP3**

| 参数 | 必填 | 类型 | 说明 | 示例 |
|------|------|------|------|------|
| `productType` | ✅ | string | `SWAP` / `SPOT` | `SWAP` |
| `page` | ✅ | integer | 页码 | `1` |
| `size` | ✅ | integer | 每页数量 | `10` |
| `sortBy` | ✅ | string | 排序字段，如 `h1net`（1h净流入） | `h1net` |
| `sortType` | ✅ | string | `desc` / `asc` | `desc` |
| `baseCoin` | ✅ | string | 币种（传空字符串查全部） | `BTC` |

#### `GET /api/fund/getFundHisList` — 历史资金流
**套餐：VIP3**

| 参数 | 必填 | 类型 | 说明 | 示例 |
|------|------|------|------|------|
| `baseCoin` | ✅ | string | 币种 | `BTC` |
| `endTime` | ✅ | number | 毫秒时间戳 | `当前时间戳` |
| `productType` | ✅ | string | `SWAP` / `SPOT` | `SWAP` |
| `size` | ✅ | integer | 数量 | `10` |
| `interval` | ✅ | string | 周期 | `1h` |

```bash
curl -H "apikey: $APIKEY" \
  "https://open-api.coinank.com/api/fund/fundReal?productType=SWAP&page=1&size=10&sortBy=h1net&sortType=desc&baseCoin=BTC"

curl -H "apikey: $APIKEY" \
  "https://open-api.coinank.com/api/fund/getFundHisList?baseCoin=BTC&endTime=$NOW&productType=SWAP&size=10&interval=1h"
```

---

### 16. 订单流

#### `GET /api/orderFlow/lists` — 订单流数据
**套餐：VIP3**

| 参数 | 必填 | 类型 | 说明 | 示例 |
|------|------|------|------|------|
| `exchange` | ✅ | string | 交易所 | `Binance` |
| `symbol` | ✅ | string | 交易对 | `BTCUSDT` |
| `interval` | ✅ | string | 周期 | `1h` |
| `endTime` | ✅ | number | 毫秒时间戳 | `当前时间戳` |
| `size` | ✅ | integer | 数量 | `10` |
| `productType` | ✅ | string | `SWAP` / `SPOT` | `SWAP` |
| `tickCount` | ✅ | integer | tick 数量 | `1` |

```bash
curl -H "apikey: $APIKEY" \
  "https://open-api.coinank.com/api/orderFlow/lists?exchange=Binance&symbol=BTCUSDT&interval=1h&endTime=$NOW&size=10&productType=SWAP&tickCount=1"
```

---

### 17. 资金费率

#### `GET /api/fundingRate/hist` — 历史资金费率（跨交易所）
**套餐：VIP1**

| 参数 | 必填 | 类型 | 说明 | 示例 |
|------|------|------|------|------|
| `baseCoin` | ✅ | string | 币种 | `BTC` |
| `exchangeType` | ✅ | string | 计价币类型：`USDT` / `USD`（币本位） | `USDT` |
| `endTime` | ✅ | number | 毫秒时间戳 | `当前时间戳` |
| `size` | ✅ | integer | 数量 | `10` |

#### `GET /api/fundingRate/current` — 实时资金费率排行
**套餐：VIP1**

| 参数 | 必填 | 类型 | 说明 | 示例 |
|------|------|------|------|------|
| `type` | ✅ | string | `current` 实时 / `day` 1日 / `week` 1周 / `month` 1月 / `year` 1年 | `current` |

#### `GET /api/fundingRate/accumulated` — 累计资金费率
**套餐：VIP1**

| 参数 | 必填 | 类型 | 说明 | 示例 |
|------|------|------|------|------|
| `type` | ✅ | string | `day` / `week` / `month` / `year` | `day` |

#### `GET /api/fundingRate/indicator` — 交易对资金费率历史
**套餐：VIP1**

| 参数 | 必填 | 类型 | 说明 | 示例 |
|------|------|------|------|------|
| `exchange` | ✅ | string | 交易所（Binance / OKX / Bybit / Huobi / Gate / Bitget） | `Binance` |
| `symbol` | ✅ | string | 交易对 | `BTCUSDT` |
| `interval` | ✅ | string | 周期 | `1h` |
| `endTime` | ✅ | string | 毫秒时间戳 | `当前时间戳` |
| `size` | ✅ | string | 数量，最大 500 | `10` |

#### `GET /api/fundingRate/kline` — 资金费率 K 线
**套餐：VIP1**

参数与 `fundingRate/indicator` 相同。

#### `GET /api/fundingRate/getWeiFr` — 加权资金费率
**套餐：VIP1**

| 参数 | 必填 | 类型 | 说明 | 示例 |
|------|------|------|------|------|
| `baseCoin` | ✅ | string | 币种 | `BTC` |
| `interval` | ✅ | string | 周期 | `1h` |
| `endTime` | ✅ | string | 毫秒时间戳 | `当前时间戳` |
| `size` | ✅ | string | 数量，最大 500 | `10` |

#### `GET /api/fundingRate/frHeatmap` — 资金费率热力图
**套餐：VIP1**

| 参数 | 必填 | 类型 | 说明 | 示例 |
|------|------|------|------|------|
| `type` | ✅ | string | `openInterest` 按持仓 / `marketCap` 按市值 | `marketCap` |
| `interval` | ✅ | string | `1D` / `1W` / `1M` / `6M` | `1M` |

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

---

### 18. RSI 选币器

#### `GET /api/rsiMap/list` — RSI 指标筛选
**套餐：VIP2**

| 参数 | 必填 | 类型 | 说明 | 示例 |
|------|------|------|------|------|
| `interval` | ✅ | string | 周期（注意大写H/D）：`1H` / `4H` / `1D` 等 | `1H` |
| `exchange` | ✅ | string | 交易所 | `Binance` |

```bash
curl -H "apikey: $APIKEY" \
  "https://open-api.coinank.com/api/rsiMap/list?interval=1H&exchange=Binance"
```

---

## 枚举值速查

### interval（K线/历史数据周期）

| 值 | 说明 |
|----|------|
| `1m` | 1 分钟 |
| `3m` | 3 分钟 |
| `5m` | 5 分钟 |
| `15m` | 15 分钟 |
| `30m` | 30 分钟 |
| `1h` | 1 小时 |
| `2h` | 2 小时 |
| `4h` | 4 小时 |
| `6h` | 6 小时 |
| `8h` | 8 小时 |
| `12h` | 12 小时 |
| `1d` | 1 天 |

> RSI 选币器使用大写：`1H`、`4H`、`1D`
> 资金费率热力图使用：`1D`、`1W`、`1M`、`6M`

### exchange（主流交易所）

| 值 | 说明 |
|----|------|
| `Binance` | 币安 |
| `OKX` | 欧易 |
| `Bybit` | Bybit |
| `Bitget` | Bitget |
| `Gate` | Gate.io |
| `Huobi` | 火币 |
| `Bitmex` | BitMEX |
| `dYdX` | dYdX |
| `Bitfinex` | Bitfinex |
| `CME` | 芝商所 |
| `Kraken` | Kraken |
| `Deribit` | Deribit |

### productType（产品类型）

| 值 | 说明 |
|----|------|
| `SWAP` | 永续合约 |
| `SPOT` | 现货 |
| `FUTURES` | 交割合约 |

### sortBy 常用字段

| 接口类型 | 常用 sortBy 值 |
|----------|---------------|
| 持仓排行 | `openInterest` |
| 爆仓排行 | `liquidationH24`、`liquidationH12`、`liquidationH8`、`liquidationH4`、`liquidationH1` |
| 价格排行 | `priceChangeH24`、`priceChangeH1`、`priceChangeM5` |
| 交易量排行 | `h24Volume`、`h1Volume` |
| 笔数排行 | `h1Count`、`d1Count`、`h4Count` |
| 资金流 | `h1net`、`h4net`、`h8net`、`h24net` |
| 鲸鱼持仓 | `positionValue`、`unrealizedPnl` |
