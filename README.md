<div align="center">

# CoinAnk OpenAPI Skill

### `> AI Agent 的加密衍生品数据引擎_`

<br />

[![Endpoints](https://img.shields.io/badge/59_Endpoints-18_Categories-00d4aa?style=for-the-badge&logo=bitcoin&logoColor=white)](#接口总览)
[![OpenClaw](https://img.shields.io/badge/OpenClaw-Skill-ff6b6b?style=for-the-badge&logo=openai&logoColor=white)](https://github.com/openclaw/openclaw)
[![REST](https://img.shields.io/badge/REST-API-3178c6?style=for-the-badge&logo=fastapi&logoColor=white)](https://open-api.coinank.com)
[![License](https://img.shields.io/badge/MIT-License-f59e0b?style=for-the-badge&logo=opensourceinitiative&logoColor=white)](./LICENSE)

<br />

[简体中文](./README.md) · [English](./README_EN.md)

<br />

<img src="https://raw.githubusercontent.com/andreasbm/readme/master/assets/lines/rainbow.png" alt="-----" />

</div>

<div align="center">

## 什么是 CoinAnk OpenAPI Skill？

**一句话查数据。18 大类衍生品指标，尽在掌握。**

</div>

<div align="center">

CoinAnk OpenAPI Skill 是一个 [OpenClaw](https://github.com/openclaw/openclaw) Skill（AI Agent 插件），为大语言模型提供完整的加密货币衍生品市场数据能力。覆盖 **K 线、ETF、持仓、多空比、资金费率、爆仓、订单流、鲸鱼动向**等 18 大类、59 个实时数据接口，全部经过实测验证可用。

</div>

<div align="center">

<table>
<tr><td>

- 全部 **59 个接口**经过实测验证
- 所有请求均为 **GET**，简洁高效
- 支持 **VIP1 ~ VIP4** 分级权限体系

</td></tr>
</table>

<br />
<img src="https://raw.githubusercontent.com/andreasbm/readme/master/assets/lines/rainbow.png" alt="-----" />

## 数据覆盖

</div>

<div align="center">
<table>
<tr>
<td width="50%">

**行情与价格**

| 分类 | 接口数 | 套餐 |
|:--|:--:|:--:|
| K 线 | 1 | VIP1 |
| ETF | 5 | VIP1 |
| 币种和交易对 | 4 | VIP1 |
| 指标数据 | 10 | VIP1 |
| 新闻快讯 | 2 | VIP2 |

</td>
<td width="50%">

**衍生品深度**

| 分类 | 接口数 | 套餐 |
|:--|:--:|:--:|
| 未平仓合约 | 7 | VIP1 |
| 资金费率 | 7 | VIP1 |
| 多空比 | 6 | VIP1 |
| 爆仓数据 | 8 | VIP1 |
| RSI 选币器 | 1 | VIP2 |

</td>
</tr>
<tr>
<td width="50%">

**机构级数据**

| 分类 | 接口数 | 套餐 |
|:--|:--:|:--:|
| 大额订单 | 2 | VIP3 |
| 市价单统计 | 8 | VIP3 |
| 订单本 | 3 | VIP3 |
| 资金流 | 2 | VIP3 |
| 订单流 | 1 | VIP3 |
| 净多头/净空头 | 1 | VIP3 |

</td>
<td width="50%">

**链上与鲸鱼**

| 分类 | 接口数 | 套餐 |
|:--|:--:|:--:|
| HyperLiquid 鲸鱼 | 2 | VIP2 |
| 热门排行 | 8 | VIP2 |

</td>
</tr>
</table>
</div>

<div align="center">

**合计：18 大类 · 59 个接口**

<br />
<img src="https://raw.githubusercontent.com/andreasbm/readme/master/assets/lines/rainbow.png" alt="-----" />

## 快速开始

</div>

```bash
# 1. 克隆到 OpenClaw Skills 目录
git clone https://github.com/coinank/coinank-openapi-skill.git ~/.openclaw/skills/coinank-openapi-skill

# 2. 在环境变量中设置 API Key
export COINANK_API_KEY="your_api_key_here"
```

<div align="center">

然后在 OpenClaw Agent 中直接用自然语言查询：

</div>

```
> "查一下 BTC 当前的资金费率"
> "看看过去 24 小时的爆仓数据"
> "获取以太坊的多空比"
```

<div align="center">
<br />
<img src="https://raw.githubusercontent.com/andreasbm/readme/master/assets/lines/rainbow.png" alt="-----" />

## 认证与请求规范

</div>

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

<div align="center">
<br />
<img src="https://raw.githubusercontent.com/andreasbm/readme/master/assets/lines/rainbow.png" alt="-----" />

## 关键注意事项

</div>

### 1. 时间戳必须是毫秒级且为当前时间

所有 `endTime` 参数均为**毫秒级时间戳**，且必须接近当前时间。传入过期或格式错误的时间戳会返回 `code: -7`。

```bash
# 正确：使用 python3 生成（跨平台兼容）
NOW=$(python3 -c "import time; print(int(time.time()*1000))")

# 错误：macOS 的 date 命令不支持 %3N，会生成如 "17228693N" 的无效值
NOW=$(date +%s%3N)  # 不要用这个！
```

### 2. 套餐权限等级

接口分为 VIP1～VIP4 四个级别，级别越高可访问的接口越多。每个接口标注了所需最低套餐。

### 3. `exchanges` 参数传空字符串

`getAggCvd`、`getAggBuySellCount` 等聚合市价单接口中，`exchanges` 参数**必须传入**（传空字符串 `exchanges=` 表示聚合所有交易所）。

### 4. OpenAPI 文件中的时间戳仅为示例

`references/` 目录下 JSON 文件中的 `example` 时间戳均为历史示例，调用时应使用实时生成的时间戳。

<div align="center">
<br />
<img src="https://raw.githubusercontent.com/andreasbm/readme/master/assets/lines/rainbow.png" alt="-----" />

## 接口详情

</div>

---

<details>
<summary><strong>1. K 线</strong> — 1 个接口 · VIP1</summary>

<br />

#### `GET /api/kline/lists` — K线行情数据

| 参数 | 必填 | 类型 | 说明 | 示例 |
|------|------|------|------|------|
| `symbol` | Yes | string | 交易对 | `BTCUSDT` |
| `exchange` | Yes | string | 交易所 | `Binance` |
| `endTime` | Yes | number | 毫秒时间戳，返回此时间之前的数据 | `当前时间戳` |
| `size` | Yes | integer | 数量，最大 500 | `10` |
| `interval` | Yes | string | 周期，见枚举值 | `1h` |
| `productType` | Yes | string | `SWAP` 合约 / `SPOT` 现货 | `SWAP` |

```bash
curl -H "apikey: $APIKEY" \
  "https://open-api.coinank.com/api/kline/lists?symbol=BTCUSDT&exchange=Binance&endTime=$NOW&size=10&interval=1h&productType=SWAP"
```

</details>

---

<details>
<summary><strong>2. ETF</strong> — 5 个接口 · VIP1</summary>

<br />

#### `GET /api/etf/getUsBtcEtf` — 美国 BTC ETF 列表
**无需参数**

#### `GET /api/etf/getUsEthEtf` — 美国 ETH ETF 列表
**无需参数**

#### `GET /api/etf/usBtcInflow` — 美国 BTC ETF 历史净流入
**无需参数**

#### `GET /api/etf/usEthInflow` — 美国 ETH ETF 历史净流入
**无需参数**

#### `GET /api/etf/hkEtfInflow` — 港股 ETF 历史净流入
**无需参数**

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
<summary><strong>3. HyperLiquid 鲸鱼</strong> — 2 个接口 · VIP2</summary>

<br />

#### `GET /api/hyper/topPosition` — 鲸鱼持仓排行

| 参数 | 必填 | 类型 | 说明 | 示例 |
|------|------|------|------|------|
| `sortBy` | Yes | string | 排序字段 | `positionValue` |
| `sortType` | Yes | string | `desc` 降序 / `asc` 升序 | `desc` |
| `page` | Yes | integer | 页码 | `1` |
| `size` | Yes | integer | 每页数量 | `10` |

#### `GET /api/hyper/topAction` — 鲸鱼最新动态
**无需参数**

```bash
curl -H "apikey: $APIKEY" \
  "https://open-api.coinank.com/api/hyper/topPosition?sortBy=positionValue&sortType=desc&page=1&size=10"

curl -H "apikey: $APIKEY" "https://open-api.coinank.com/api/hyper/topAction"
```

</details>

---

<details>
<summary><strong>4. 净多头和净空头</strong> — 1 个接口 · VIP3</summary>

<br />

#### `GET /api/netPositions/getNetPositions` — 净多头/净空头历史

| 参数 | 必填 | 类型 | 说明 | 示例 |
|------|------|------|------|------|
| `exchange` | Yes | string | 交易所 | `Binance` |
| `symbol` | Yes | string | 交易对 | `BTCUSDT` |
| `interval` | Yes | string | 周期 | `1h` |
| `endTime` | Yes | number | 毫秒时间戳 | `当前时间戳` |
| `size` | Yes | integer | 数量，最大 500 | `10` |

```bash
curl -H "apikey: $APIKEY" \
  "https://open-api.coinank.com/api/netPositions/getNetPositions?exchange=Binance&symbol=BTCUSDT&interval=1h&endTime=$NOW&size=10"
```

</details>

---

<details>
<summary><strong>5. 大额订单</strong> — 2 个接口 · VIP3</summary>

<br />

#### `GET /api/trades/largeTrades` — 大额市价订单

| 参数 | 必填 | 类型 | 说明 | 示例 |
|------|------|------|------|------|
| `symbol` | Yes | string | 交易对 | `BTCUSDT` |
| `productType` | Yes | string | `SWAP` 合约 / `SPOT` 现货 | `SWAP` |
| `amount` | Yes | string | 最小金额（USD） | `10000000` |
| `endTime` | Yes | string | 毫秒时间戳 | `当前时间戳` |
| `size` | Yes | string | 数量，最大 500 | `10` |

#### `GET /api/bigOrder/queryOrderList` — 大额限价订单

| 参数 | 必填 | 类型 | 说明 | 示例 |
|------|------|------|------|------|
| `symbol` | Yes | string | 交易对 | `BTCUSDT` |
| `exchangeType` | Yes | string | `SWAP` 永续 / `SPOT` 现货 / `FUTURES` 交割 | `SWAP` |
| `size` | Yes | integer | 数量，最大 500 | `10` |
| `amount` | Yes | number | 最低金额（USD） | `1000000` |
| `side` | Yes | string | `ask` 卖 / `bid` 买 | `ask` |
| `exchange` | Yes | string | 交易所（Binance / OKX / Coinbase） | `Binance` |
| `isHistory` | Yes | string | `true` 历史 / `false` 实时 | `true` |
| `startTime` | No | number | 截止时间戳（isHistory=true 时建议传当前时间戳） | `当前时间戳` |

```bash
curl -H "apikey: $APIKEY" \
  "https://open-api.coinank.com/api/trades/largeTrades?symbol=BTCUSDT&productType=SWAP&amount=10000000&endTime=$NOW&size=10"

curl -H "apikey: $APIKEY" \
  "https://open-api.coinank.com/api/bigOrder/queryOrderList?symbol=BTCUSDT&exchangeType=SWAP&size=10&amount=1000000&side=ask&exchange=Binance&isHistory=true&startTime=$NOW"
```

</details>

---

<details>
<summary><strong>6. 币种和交易对</strong> — 4 个接口 · VIP1</summary>

<br />

#### `GET /api/instruments/getLastPrice` — 实时价格

| 参数 | 必填 | 类型 | 说明 | 示例 |
|------|------|------|------|------|
| `symbol` | Yes | string | 交易对 | `BTCUSDT` |
| `exchange` | Yes | string | 交易所 | `Binance` |
| `productType` | Yes | string | `SWAP` / `SPOT` | `SWAP` |

#### `GET /api/instruments/getCoinMarketCap` — 币种市值信息

| 参数 | 必填 | 类型 | 说明 | 示例 |
|------|------|------|------|------|
| `baseCoin` | Yes | string | 币种 | `BTC` |

#### `GET /api/baseCoin/list` — 支持的币种列表

| 参数 | 必填 | 类型 | 说明 | 示例 |
|------|------|------|------|------|
| `productType` | Yes | string | `SWAP` / `SPOT` | `SWAP` |

#### `GET /api/baseCoin/symbols` — 支持的交易对列表

| 参数 | 必填 | 类型 | 说明 | 示例 |
|------|------|------|------|------|
| `exchange` | Yes | string | 交易所 | `Binance` |
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
<summary><strong>7. 多空比</strong> — 6 个接口 · VIP1</summary>

<br />

#### `GET /api/longshort/buySell` — 全市场多空买卖比
**套餐：VIP3**

| 参数 | 必填 | 类型 | 说明 | 示例 |
|------|------|------|------|------|
| `baseCoin` | Yes | string | 币种 | `BTC` |
| `interval` | Yes | string | 周期 | `1h` |
| `endTime` | Yes | string | 毫秒时间戳 | `当前时间戳` |
| `size` | Yes | string | 数量 | `10` |

#### `GET /api/longshort/realtimeAll` — 交易所实时多空比

| 参数 | 必填 | 类型 | 说明 | 示例 |
|------|------|------|------|------|
| `baseCoin` | Yes | string | 币种 | `BTC` |
| `interval` | Yes | string | 周期，可选 `5m/15m/30m/1h/2h/4h/6h/8h/12h/1d` | `1h` |

#### `GET /api/longshort/person` — 多空持仓人数比
**支持交易所：Binance / OKX / Bybit**

| 参数 | 必填 | 类型 | 说明 | 示例 |
|------|------|------|------|------|
| `exchange` | Yes | string | 交易所 | `Binance` |
| `symbol` | Yes | string | 交易对 | `BTCUSDT` |
| `interval` | Yes | string | 周期 | `1h` |
| `endTime` | Yes | string | 毫秒时间戳 | `当前时间戳` |
| `size` | Yes | string | 数量，最大 500 | `10` |

#### `GET /api/longshort/position` — 大户多空比（持仓量）
**支持交易所：Binance / OKX / Huobi** — 参数与 `person` 相同。

#### `GET /api/longshort/account` — 大户多空比（账户数）
**支持交易所：Binance / OKX / Huobi** — 参数与 `person` 相同。

#### `GET /api/longshort/kline` — 多空比 K 线
**支持交易所：Binance / OKX / Huobi**

| 参数 | 必填 | 类型 | 说明 | 示例 |
|------|------|------|------|------|
| `exchange` | Yes | string | 交易所 | `Binance` |
| `symbol` | Yes | string | 交易对 | `BTCUSDT` |
| `interval` | Yes | string | 周期 | `1h` |
| `endTime` | Yes | string | 毫秒时间戳 | `当前时间戳` |
| `size` | Yes | string | 数量，最大 500 | `10` |
| `type` | Yes | string | `longShortPerson` 人数比 / `longShortPosition` 持仓比 / `longShortAccount` 账户比 | `longShortPerson` |

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
<summary><strong>8. 市价单统计指标</strong> — 8 个接口 · VIP3</summary>

<br />

> 分为**单交易对**和**聚合（跨交易所）**两组。

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
| `exchange` | Yes | string | 交易所（Binance / OKX / Bybit / Bitget） | `Binance` |
| `symbol` | Yes | string | 交易对 | `BTCUSDT` |
| `interval` | Yes | string | 周期 | `1h` |
| `endTime` | Yes | string | 毫秒时间戳 | `当前时间戳` |
| `size` | Yes | integer | 数量，最大 500 | `10` |
| `productType` | Yes | string | `SWAP` / `SPOT` | `SWAP` |

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
| `baseCoin` | Yes | string | 币种 | `BTC` |
| `interval` | Yes | string | 周期 | `1h` |
| `endTime` | Yes | string | 毫秒时间戳 | `当前时间戳` |
| `size` | Yes | integer | 数量，最大 500 | `10` |
| `productType` | Yes | string | `SWAP` / `SPOT` | `SWAP` |
| `exchanges` | Yes | string | **传空字符串**表示聚合所有交易所 | `（空）` |

```bash
curl -H "apikey: $APIKEY" \
  "https://open-api.coinank.com/api/marketOrder/getCvd?exchange=Binance&symbol=BTCUSDT&interval=1h&endTime=$NOW&size=10&productType=SWAP"

# 注意：exchanges 参数必须传入，传空字符串表示聚合全部
curl -H "apikey: $APIKEY" \
  "https://open-api.coinank.com/api/marketOrder/getAggCvd?baseCoin=BTC&interval=1h&endTime=$NOW&size=10&productType=SWAP&exchanges="
```

</details>

---

<details>
<summary><strong>9. 新闻快讯</strong> — 2 个接口 · VIP2</summary>

<br />

#### `GET /api/news/getNewsList` — 新闻/快讯列表

| 参数 | 必填 | 类型 | 说明 | 示例 |
|------|------|------|------|------|
| `type` | Yes | string | `1` 快讯 / `2` 新闻 | `1` |
| `lang` | Yes | string | 语言：`zh` 中文 / `en` 英文 | `zh` |
| `page` | Yes | string | 页码 | `1` |
| `pageSize` | Yes | string | 每页数量 | `10` |
| `isPopular` | Yes | string | 是否推荐：`true` / `false` | `false` |
| `search` | Yes | string | 搜索关键词，无则传空字符串 | `（空）` |

#### `GET /api/news/getNewsDetail` — 新闻详情

| 参数 | 必填 | 类型 | 说明 | 示例 |
|------|------|------|------|------|
| `id` | Yes | string | 新闻 ID（从列表接口获取） | `69a2f40912d08f6a781aedd0` |

```bash
curl -H "apikey: $APIKEY" \
  "https://open-api.coinank.com/api/news/getNewsList?type=1&lang=zh&page=1&pageSize=10&isPopular=false&search="

curl -H "apikey: $APIKEY" \
  "https://open-api.coinank.com/api/news/getNewsDetail?id=69a2f40912d08f6a781aedd0"
```

</details>

---

<details>
<summary><strong>10. 指标数据</strong> — 10 个接口 · VIP1</summary>

<br />

> 以下指标无需参数，直接请求即可。

| 接口 | 说明 |
|------|------|
| `GET /api/indicator/getBtcMultiplier` | 两年 MA 乘数 |
| `GET /api/indicator/getCnnEntity` | 贪婪恐惧指数 |
| `GET /api/indicator/getAhr999` | ahr999 囤币指标 |
| `GET /api/indicator/getPuellMultiple` | 普尔系数（Puell Multiple） |
| `GET /api/indicator/getBtcPi` | Pi 循环顶部指标 |
| `GET /api/indicator/getMovingAvgHeatmap` | 200 周均线热力图 |
| `GET /api/indicator/getAltcoinSeason` | 山寨季指数 |

#### `GET /api/indicator/getMarketCapRank` — 市值占比排名

| 参数 | 必填 | 类型 | 说明 | 示例 |
|------|------|------|------|------|
| `symbol` | Yes | string | 币种 | `BTC` |

#### `GET /api/indicator/getGrayscaleOpenInterest` — 灰度持仓数据

| 参数 | 必填 | 类型 | 说明 | 示例 |
|------|------|------|------|------|
| `symbol` | Yes | string | 币种 | `BTC` |

#### `GET /api/indicator/index/charts` — 彩虹图等综合指标

| 参数 | 必填 | 类型 | 说明 | 示例 |
|------|------|------|------|------|
| `type` | Yes | string | 图表类型 | `bitcoin-rainbow-v2` |

```bash
curl -H "apikey: $APIKEY" "https://open-api.coinank.com/api/indicator/getCnnEntity"
curl -H "apikey: $APIKEY" "https://open-api.coinank.com/api/indicator/getMarketCapRank?symbol=BTC"
curl -H "apikey: $APIKEY" "https://open-api.coinank.com/api/indicator/index/charts?type=bitcoin-rainbow-v2"
```

</details>

---

<details>
<summary><strong>11. 未平仓合约</strong> — 7 个接口 · VIP1</summary>

<br />

#### `GET /api/openInterest/all` — 实时持仓列表（全交易所）

| 参数 | 必填 | 类型 | 说明 | 示例 |
|------|------|------|------|------|
| `baseCoin` | Yes | string | 币种 | `BTC` |

#### `GET /api/openInterest/v2/chart` — 币种聚合持仓历史

| 参数 | 必填 | 类型 | 说明 | 示例 |
|------|------|------|------|------|
| `baseCoin` | Yes | string | 币种 | `BTC` |
| `exchange` | Yes | string | 交易所，传空字符串查全部 | `（空）` |
| `interval` | Yes | string | 周期 | `1h` |
| `size` | Yes | string | 数量，最大 500 | `10` |
| `type` | Yes | string | `USD` 美元计价 / 币种名（如 `BTC`）币本位 | `USD` |
| `endTime` | Yes | number | 毫秒时间戳 | `当前时间戳` |

#### `GET /api/openInterest/symbol/Chart` — 交易对持仓历史

| 参数 | 必填 | 类型 | 说明 | 示例 |
|------|------|------|------|------|
| `exchange` | Yes | string | 交易所 | `Binance` |
| `symbol` | Yes | string | 交易对 | `BTCUSDT` |
| `interval` | Yes | string | 周期 | `1h` |
| `endTime` | Yes | number | 毫秒时间戳 | `当前时间戳` |
| `size` | Yes | string | 数量，最大 500 | `10` |
| `type` | Yes | string | `USD` / 币种名 | `USD` |

#### `GET /api/openInterest/kline` — 交易对持仓 K 线

| 参数 | 必填 | 类型 | 说明 | 示例 |
|------|------|------|------|------|
| `exchange` | Yes | string | 交易所 | `Binance` |
| `symbol` | Yes | string | 交易对 | `BTCUSDT` |
| `interval` | Yes | string | 周期 | `1h` |
| `endTime` | Yes | number | 毫秒时间戳 | `当前时间戳` |
| `size` | Yes | integer | 数量 | `10` |

#### `GET /api/openInterest/aggKline` — 聚合持仓 K 线

| 参数 | 必填 | 类型 | 说明 | 示例 |
|------|------|------|------|------|
| `baseCoin` | Yes | string | 币种 | `BTC` |
| `interval` | Yes | string | 周期 | `1h` |
| `endTime` | Yes | number | 毫秒时间戳 | `当前时间戳` |
| `size` | Yes | integer | 数量 | `10` |

#### `GET /api/tickers/topOIByEx` — 实时持仓（按交易所统计）

| 参数 | 必填 | 类型 | 说明 | 示例 |
|------|------|------|------|------|
| `baseCoin` | Yes | string | 币种 | `BTC` |

#### `GET /api/instruments/oiVsMc` — 历史持仓市值比
**套餐：VIP2**

| 参数 | 必填 | 类型 | 说明 | 示例 |
|------|------|------|------|------|
| `baseCoin` | Yes | string | 币种 | `BTC` |
| `endTime` | Yes | string | 毫秒时间戳 | `当前时间戳` |
| `size` | Yes | string | 数量，最大 500 | `100` |
| `interval` | Yes | string | 周期 | `1h` |

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
<summary><strong>12. 热门排行</strong> — 8 个接口 · VIP2</summary>

<br />

#### `GET /api/instruments/visualScreener` — 视觉筛选器

| 参数 | 必填 | 类型 | 说明 | 示例 |
|------|------|------|------|------|
| `interval` | Yes | string | `15m` / `1h` / `4h` / `24h` | `15m` |

#### `GET /api/instruments/oiVsMarketCap` — 持仓/市值排行

| 参数 | 必填 | 类型 | 说明 | 示例 |
|------|------|------|------|------|
| `page` | Yes | integer | 页码 | `1` |
| `size` | Yes | integer | 每页数量 | `10` |
| `sortBy` | Yes | string | 排序字段 | `openInterest` |
| `sortType` | Yes | string | `desc` / `asc` | `desc` |

#### `GET /api/instruments/longShortRank` — 多空持仓人数比排行

| 参数 | 必填 | 类型 | 说明 | 示例 |
|------|------|------|------|------|
| `sortBy` | Yes | string | 排序字段 | `longRatio` |
| `sortType` | Yes | string | `desc` / `asc` | `desc` |
| `size` | Yes | integer | 每页数量 | `10` |
| `page` | Yes | integer | 页码 | `1` |

#### `GET /api/instruments/oiRank` — 持仓量排行榜
参数同 `longShortRank`，`sortBy` 示例值：`openInterest`。

#### `GET /api/trades/count` — 交易笔数排行

| 参数 | 必填 | 类型 | 说明 | 示例 |
|------|------|------|------|------|
| `productType` | Yes | string | `SWAP` / `SPOT` | `SWAP` |
| `sortBy` | Yes | string | 排序字段，如 `h1Count`（1小时）、`d1Count`（1天） | `h1Count` |
| `sortType` | Yes | string | `desc` / `asc` | `desc` |

#### `GET /api/instruments/liquidationRank` — 爆仓排行榜

| 参数 | 必填 | 类型 | 说明 | 示例 |
|------|------|------|------|------|
| `sortBy` | Yes | string | 排序字段，如 `liquidationH24` | `liquidationH24` |
| `sortType` | Yes | string | `desc` / `asc` | `desc` |
| `page` | Yes | integer | 页码 | `1` |
| `size` | Yes | integer | 每页数量 | `10` |

#### `GET /api/instruments/priceRank` — 价格变化排行

| 参数 | 必填 | 类型 | 说明 | 示例 |
|------|------|------|------|------|
| `sortBy` | Yes | string | 如 `priceChangeH24`（24h涨跌幅） | `priceChangeH24` |
| `sortType` | Yes | string | `desc` / `asc` | `desc` |

#### `GET /api/instruments/volumeRank` — 交易量变化排行

| 参数 | 必填 | 类型 | 说明 | 示例 |
|------|------|------|------|------|
| `sortBy` | Yes | string | 如 `h24Volume`（24h交易量） | `h24Volume` |
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
<summary><strong>13. 爆仓数据</strong> — 8 个接口 · VIP1</summary>

<br />

#### `GET /api/liquidation/allExchange/intervals` — 各时间段实时爆仓统计

| 参数 | 必填 | 类型 | 说明 | 示例 |
|------|------|------|------|------|
| `baseCoin` | Yes | string | 币种 | `BTC` |

#### `GET /api/liquidation/aggregated-history` — 聚合爆仓历史

| 参数 | 必填 | 类型 | 说明 | 示例 |
|------|------|------|------|------|
| `baseCoin` | Yes | string | 币种 | `BTC` |
| `interval` | Yes | string | 周期 | `1h` |
| `endTime` | Yes | number | 毫秒时间戳 | `当前时间戳` |
| `size` | Yes | integer | 数量 | `10` |

#### `GET /api/liquidation/history` — 交易对爆仓历史

| 参数 | 必填 | 类型 | 说明 | 示例 |
|------|------|------|------|------|
| `exchange` | Yes | string | 交易所 | `Binance` |
| `symbol` | Yes | string | 交易对 | `BTCUSDT` |
| `interval` | Yes | string | 周期 | `1h` |
| `endTime` | Yes | number | 毫秒时间戳 | `当前时间戳` |
| `size` | Yes | integer | 数量 | `10` |

#### `GET /api/liquidation/orders` — 爆仓订单列表
**套餐：VIP3**

| 参数 | 必填 | 类型 | 说明 | 示例 |
|------|------|------|------|------|
| `baseCoin` | Yes | string | 币种 | `BTC` |
| `exchange` | Yes | string | 交易所 | `Binance` |
| `side` | Yes | string | `long` 多 / `short` 空 | `long` |
| `amount` | Yes | number | 最低爆仓金额（USD） | `100` |
| `endTime` | Yes | number | 毫秒时间戳 | `当前时间戳` |

#### `GET /api/liqMap/getLiqMap` — 清算地图
**套餐：VIP4**

| 参数 | 必填 | 类型 | 说明 | 示例 |
|------|------|------|------|------|
| `symbol` | Yes | string | 交易对 | `BTCUSDT` |
| `exchange` | Yes | string | 交易所 | `Binance` |
| `interval` | Yes | string | 周期 | `1d` |

#### `GET /api/liqMap/getAggLiqMap` — 聚合清算地图
**套餐：VIP4**

| 参数 | 必填 | 类型 | 说明 | 示例 |
|------|------|------|------|------|
| `baseCoin` | Yes | string | 币种 | `BTC` |
| `interval` | Yes | string | 周期 | `1d` |

#### `GET /api/liqMap/getLiqHeatMap` — 清算热力图
**套餐：VIP4**

| 参数 | 必填 | 类型 | 说明 | 示例 |
|------|------|------|------|------|
| `exchange` | Yes | string | 交易所 | `Binance` |
| `symbol` | Yes | string | 交易对 | `BTCUSDT` |
| `interval` | Yes | string | 周期 | `1d` |

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

</details>

---

<details>
<summary><strong>14. 订单本</strong> — 3 个接口 · VIP3</summary>

<br />

#### `GET /api/orderBook/v2/bySymbol` — 按交易对查询挂单深度历史

| 参数 | 必填 | 类型 | 说明 | 示例 |
|------|------|------|------|------|
| `symbol` | Yes | string | 交易对 | `BTCUSDT` |
| `exchange` | Yes | string | 交易所 | `Binance` |
| `rate` | Yes | number | 价格精度比例 | `0.01` |
| `productType` | Yes | string | `SWAP` / `SPOT` | `SWAP` |
| `interval` | Yes | string | 周期 | `1h` |
| `endTime` | Yes | number | 毫秒时间戳 | `当前时间戳` |
| `size` | Yes | integer | 数量，最大 500 | `10` |

#### `GET /api/orderBook/v2/byExchange` — 按交易所查询挂单深度历史

| 参数 | 必填 | 类型 | 说明 | 示例 |
|------|------|------|------|------|
| `baseCoin` | Yes | string | 币种 | `BTC` |
| `productType` | Yes | string | `SWAP` / `SPOT` | `SWAP` |
| `interval` | Yes | string | 周期 | `1h` |
| `endTime` | Yes | number | 毫秒时间戳 | `当前时间戳` |
| `size` | Yes | integer | 数量，最大 500 | `10` |
| `exchanges` | Yes | string | 交易所名称 | `Binance` |
| `type` | Yes | string | 价格精度比例 | `0.01` |

#### `GET /api/orderBook/getHeatMap` — 挂单流动性热力图
**套餐：VIP4**

> 此接口 `endTime` 参数会被 CDN 缓存层校验，必须传入当前毫秒时间戳。

| 参数 | 必填 | 类型 | 说明 | 示例 |
|------|------|------|------|------|
| `exchange` | Yes | string | 交易所（目前仅支持 Binance） | `Binance` |
| `symbol` | Yes | string | 交易对 | `BTCUSDT` |
| `interval` | Yes | string | 周期：`1m` / `3m` / `5m` | `1m` |
| `endTime` | Yes | string | 毫秒时间戳（**必须传当前时间**，过期时间会被 CDN 拦截返回 401） | `当前时间戳` |
| `size` | Yes | string | 数量，最大 500 | `10` |

```bash
curl -H "apikey: $APIKEY" \
  "https://open-api.coinank.com/api/orderBook/v2/bySymbol?symbol=BTCUSDT&exchange=Binance&rate=0.01&productType=SWAP&interval=1h&endTime=$NOW&size=10"

curl -H "apikey: $APIKEY" \
  "https://open-api.coinank.com/api/orderBook/getHeatMap?exchange=Binance&symbol=BTCUSDT&interval=1m&endTime=$NOW&size=10"
```

</details>

---

<details>
<summary><strong>15. 资金流</strong> — 2 个接口 · VIP3</summary>

<br />

#### `GET /api/fund/fundReal` — 实时资金流

| 参数 | 必填 | 类型 | 说明 | 示例 |
|------|------|------|------|------|
| `productType` | Yes | string | `SWAP` / `SPOT` | `SWAP` |
| `page` | Yes | integer | 页码 | `1` |
| `size` | Yes | integer | 每页数量 | `10` |
| `sortBy` | Yes | string | 排序字段，如 `h1net`（1h净流入） | `h1net` |
| `sortType` | Yes | string | `desc` / `asc` | `desc` |
| `baseCoin` | Yes | string | 币种（传空字符串查全部） | `BTC` |

#### `GET /api/fund/getFundHisList` — 历史资金流

| 参数 | 必填 | 类型 | 说明 | 示例 |
|------|------|------|------|------|
| `baseCoin` | Yes | string | 币种 | `BTC` |
| `endTime` | Yes | number | 毫秒时间戳 | `当前时间戳` |
| `productType` | Yes | string | `SWAP` / `SPOT` | `SWAP` |
| `size` | Yes | integer | 数量 | `10` |
| `interval` | Yes | string | 周期 | `1h` |

```bash
curl -H "apikey: $APIKEY" \
  "https://open-api.coinank.com/api/fund/fundReal?productType=SWAP&page=1&size=10&sortBy=h1net&sortType=desc&baseCoin=BTC"

curl -H "apikey: $APIKEY" \
  "https://open-api.coinank.com/api/fund/getFundHisList?baseCoin=BTC&endTime=$NOW&productType=SWAP&size=10&interval=1h"
```

</details>

---

<details>
<summary><strong>16. 订单流</strong> — 1 个接口 · VIP3</summary>

<br />

#### `GET /api/orderFlow/lists` — 订单流数据

| 参数 | 必填 | 类型 | 说明 | 示例 |
|------|------|------|------|------|
| `exchange` | Yes | string | 交易所 | `Binance` |
| `symbol` | Yes | string | 交易对 | `BTCUSDT` |
| `interval` | Yes | string | 周期 | `1h` |
| `endTime` | Yes | number | 毫秒时间戳 | `当前时间戳` |
| `size` | Yes | integer | 数量 | `10` |
| `productType` | Yes | string | `SWAP` / `SPOT` | `SWAP` |
| `tickCount` | Yes | integer | tick 数量 | `1` |

```bash
curl -H "apikey: $APIKEY" \
  "https://open-api.coinank.com/api/orderFlow/lists?exchange=Binance&symbol=BTCUSDT&interval=1h&endTime=$NOW&size=10&productType=SWAP&tickCount=1"
```

</details>

---

<details>
<summary><strong>17. 资金费率</strong> — 7 个接口 · VIP1</summary>

<br />

#### `GET /api/fundingRate/hist` — 历史资金费率（跨交易所）

| 参数 | 必填 | 类型 | 说明 | 示例 |
|------|------|------|------|------|
| `baseCoin` | Yes | string | 币种 | `BTC` |
| `exchangeType` | Yes | string | 计价币类型：`USDT` / `USD`（币本位） | `USDT` |
| `endTime` | Yes | number | 毫秒时间戳 | `当前时间戳` |
| `size` | Yes | integer | 数量 | `10` |

#### `GET /api/fundingRate/current` — 实时资金费率排行

| 参数 | 必填 | 类型 | 说明 | 示例 |
|------|------|------|------|------|
| `type` | Yes | string | `current` 实时 / `day` 1日 / `week` 1周 / `month` 1月 / `year` 1年 | `current` |

#### `GET /api/fundingRate/accumulated` — 累计资金费率

| 参数 | 必填 | 类型 | 说明 | 示例 |
|------|------|------|------|------|
| `type` | Yes | string | `day` / `week` / `month` / `year` | `day` |

#### `GET /api/fundingRate/indicator` — 交易对资金费率历史

| 参数 | 必填 | 类型 | 说明 | 示例 |
|------|------|------|------|------|
| `exchange` | Yes | string | 交易所（Binance / OKX / Bybit / Huobi / Gate / Bitget） | `Binance` |
| `symbol` | Yes | string | 交易对 | `BTCUSDT` |
| `interval` | Yes | string | 周期 | `1h` |
| `endTime` | Yes | string | 毫秒时间戳 | `当前时间戳` |
| `size` | Yes | string | 数量，最大 500 | `10` |

#### `GET /api/fundingRate/kline` — 资金费率 K 线
参数与 `fundingRate/indicator` 相同。

#### `GET /api/fundingRate/getWeiFr` — 加权资金费率

| 参数 | 必填 | 类型 | 说明 | 示例 |
|------|------|------|------|------|
| `baseCoin` | Yes | string | 币种 | `BTC` |
| `interval` | Yes | string | 周期 | `1h` |
| `endTime` | Yes | string | 毫秒时间戳 | `当前时间戳` |
| `size` | Yes | string | 数量，最大 500 | `10` |

#### `GET /api/fundingRate/frHeatmap` — 资金费率热力图

| 参数 | 必填 | 类型 | 说明 | 示例 |
|------|------|------|------|------|
| `type` | Yes | string | `openInterest` 按持仓 / `marketCap` 按市值 | `marketCap` |
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
<summary><strong>18. RSI 选币器</strong> — 1 个接口 · VIP2</summary>

<br />

#### `GET /api/rsiMap/list` — RSI 指标筛选

| 参数 | 必填 | 类型 | 说明 | 示例 |
|------|------|------|------|------|
| `interval` | Yes | string | 周期（注意大写H/D）：`1H` / `4H` / `1D` 等 | `1H` |
| `exchange` | Yes | string | 交易所 | `Binance` |

```bash
curl -H "apikey: $APIKEY" \
  "https://open-api.coinank.com/api/rsiMap/list?interval=1H&exchange=Binance"
```

</details>

<div align="center">
<br />
<img src="https://raw.githubusercontent.com/andreasbm/readme/master/assets/lines/rainbow.png" alt="-----" />

## 枚举值速查

</div>

<details>
<summary><strong>interval（K线/历史数据周期）</strong></summary>

<br />

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

</details>

<details>
<summary><strong>exchange（主流交易所）</strong></summary>

<br />

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

</details>

<details>
<summary><strong>productType（产品类型）</strong></summary>

<br />

| 值 | 说明 |
|----|------|
| `SWAP` | 永续合约 |
| `SPOT` | 现货 |
| `FUTURES` | 交割合约 |

</details>

<details>
<summary><strong>sortBy 常用字段</strong></summary>

<br />

| 接口类型 | 常用 sortBy 值 |
|----------|---------------|
| 持仓排行 | `openInterest` |
| 爆仓排行 | `liquidationH24`、`liquidationH12`、`liquidationH8`、`liquidationH4`、`liquidationH1` |
| 价格排行 | `priceChangeH24`、`priceChangeH1`、`priceChangeM5` |
| 交易量排行 | `h24Volume`、`h1Volume` |
| 笔数排行 | `h1Count`、`d1Count`、`h4Count` |
| 资金流 | `h1net`、`h4net`、`h8net`、`h24net` |
| 鲸鱼持仓 | `positionValue`、`unrealizedPnl` |

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
