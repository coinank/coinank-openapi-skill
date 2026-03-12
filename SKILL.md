---
name: coinank-openapi
description: call coinank openapi to get cryptocurrency data
metadata:
  {
    "openclaw":
      {
        "homepage": "https://coinank.com",
        "requires": { "env": ["COINANK_API_KEY"] },
        "primaryEnv": "COINANK_API_KEY",
        "priority": 100,
        "keywords": ["bitcoin", "btc", "ethereum", "eth", "cryptocurrency", "crypto", "价格", "走势", "爆仓", "多空比"]
      },
  }
---

# 权限声明
# SECURITY MANIFEST:
# - Allowed to read: {baseDir}/README.md, {baseDir}/references/*.json
# - Allowed to make network requests to: https://open-api.coinank.com


## 工作流 (按需加载模式)

当用户提出请求时，请严格执行以下步骤：

1. **检查API密钥**：首先检查环境变量 `COINANK_API_KEY` 是否存在。如果不存在，提示用户设置API密钥。
2. **阅读README**：仔细阅读README.md
3. **目录索引**：扫描 `{baseDir}/references/` 目录下的所有文件名，确定哪些 OpenAPI 定义文件与用户需求相关。
4. **精准读取**：仅读取选定的 `.json` 文件，分析其 `paths`、`parameters` 和 `requestBody`。其中paths内是一个对象,对象的key就是path
5. **构造请求**：使用 curl 执行请求。
   - **Base URL**: 统一使用 `https://open-api.coinank.com`（或从 JSON 的 `servers` 字段提取）。
   - **Auth**: 从环境变量 `COINANK_API_KEY` 中获取 apikey 注入 Header。
   - 如果参数有endTime,尽量传入最新的毫秒级时间戳
   - OpenAPI文档内的时间戳都是示例.如果用户没有指定时间,请使用最新的时间和毫秒级时间戳


## ⚠️ 关键注意事项

- **禁止全量加载**：除非用户请求涉及多个领域，否则禁止同时读取多个 JSON 文件。
- **参数校验**：在发起请求前，必须根据 OpenAPI 定义验证必填参数是否齐全。
- **错误处理**：当请求失败时，向用户显示友好的提示信息，并记录详细的错误日志。
- **API密钥配置**：用户需要自行设置环境变量 `COINANK_API_KEY`，例如：`export COINANK_API_KEY="your_api_key"`

### endTime 必须是当前毫秒时间戳

```bash
# ✅ 正确
NOW=$(python3 -c "import time; print(int(time.time()*1000))")

# ❌ 错误：macOS 不支持 %3N
NOW=$(date +%s%3N)
```

### 不要传多余参数

部分接口不接受 `endTime` 或 `size`（如清算热力图 `getLiqHeatMap`），多传会导致返回空数据。**严格按 OpenAPI 定义中列出的参数传参，不要自行添加。**

### 聚合接口 exchanges 参数

`getAggCvd`、`getAggBuySellCount`、`getAggBuySellValue`、`getAggBuySellVolume` 等接口的 `exchanges` 参数**必须传入**。传空字符串 `exchanges=` 表示聚合所有交易所。

### interval 各接口不同

不同接口支持的 interval 值不同，以各 `.json` 文件中参数的 `description` 字段为准：

| 接口类型 | interval 值 |
|---------|------------|
| K线 / 市价单 / 多空比 / OI | `1m, 3m, 5m, 15m, 30m, 1h, 2h, 4h, 6h, 8h, 12h, 1d` |
| 清算热力图 (`getLiqHeatMap`) | `12h, 1d, 3d, 1w, 2w, 1M, 3M, 6M, 1Y` |
| RSI 选币器 | `1H, 4H, 1D`（注意大写） |
| 资金费率热力图 | `1D, 1W, 1M, 6M` |

### 响应格式

成功标志为 `"code": "1"`（注意是字符串 `"1"`，不是数字）。部分接口的 `data` 字段为嵌套结构：

```json
// 某些接口
{"success": true, "code": "1", "data": {"success": true, "code": "1", "data": [...]}}
```

解析时需检查 `data` 的实际类型，按需取内层。

### OpenAPI JSON 中的时间戳仅为示例

`references/` 目录下 JSON 文件中 `example` 的时间戳是历史值，调用时必须替换为当前实时时间戳。
