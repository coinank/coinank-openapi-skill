---
name: coinank-openapi
description: call coinank openapi to get data
metadata:
  {
    "openclaw":
      {
        "homepage": "https://coinank.com",
        "requires": { "env": ["COINANK_API_KEY"] },
        "primaryEnv": "COINANK_API_KEY",
      },
  }
---

# 权限声明
# SECURITY MANIFEST:
# - Allowed to read: {baseDir}/references/*.json
# - Allowed to make network requests to: https://open-api.coinank.com


## 工作流 (按需加载模式)

当用户提出请求时，请严格执行以下步骤：

1. **目录索引**：首先扫描 `{baseDir}/references/` 目录下的所有文件名，确定哪些 OpenAPI 定义文件与用户需求相关。
2. **精准读取**：仅读取选定的 `.json` 文件，分析其 `paths`、`parameters` 和 `requestBody`。其中paths内是一个对象,对象的key就是path
3. **构造请求**：使用 curl 执行请求。
   - **Base URL**: 统一使用 `https://open-api.coinank.com`（或从 JSON 的 `servers` 字段提取）。
   - **Auth**: 从环境变量 `COINANK_API_KEY` 中获取 apikey 注入 Header。
   - 如果参数有endTime,尽量传入最新的毫秒级时间戳
   - OpenAPI文档内的时间戳都是示例.如果用户没有指定时间,请使用最新的时间和毫秒级时间戳


## 注意事项
- **禁止全量加载**：除非用户请求涉及多个领域，否则禁止同时读取多个 JSON 文件。
- **参数校验**：在发起请求前，必须根据 OpenAPI 定义验证必填参数是否齐全。