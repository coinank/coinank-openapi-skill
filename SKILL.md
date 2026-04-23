---
name: coinank-openapi
description: Access CoinAnk OpenAPI for cryptocurrency market and derivatives data
metadata:
  {
    "openclaw":
      {
        "homepage": "https://coinank.com",
        "requires": { "env": ["COINANK_API_KEY"] },
        "primaryEnv": "COINANK_API_KEY",
        "priority": 100,
        "keywords": ["bitcoin", "btc", "ethereum", "eth", "cryptocurrency", "crypto", "price", "trend", "liquidation", "long-short ratio"]
      },
  }
---

# Permissions
# SECURITY MANIFEST:
# - Allowed to read: {baseDir}/README.md, {baseDir}/references/*.json
# - Allowed to make network requests to: https://open-api.coinank.com


## Operating Mode

This skill must operate in an on-demand loading mode. Do not read every OpenAPI file by default. Load only the documentation required for the user's request.


## Required Workflow

When handling a user request, follow this sequence strictly:

1. **Verify the API key**
   Check whether the `COINANK_API_KEY` environment variable is available. If it is missing, instruct the user to set it before continuing.

2. **Read the project README**
   Read `README.md` before constructing any request so you follow the documented conventions and edge cases.

3. **Identify the relevant API category**
   Scan the filenames under `{baseDir}/references/` and determine which OpenAPI file is relevant to the user's request.

4. **Read only the required schema**
   Open only the selected `.json` file and inspect its `paths`, parameters, and response definitions. In the `paths` object, each key is an API path.

5. **Validate request parameters**
   Confirm the required parameters, supported values, and whether the target endpoint accepts optional parameters such as `endTime`, `size`, `interval`, or `exchanges`.

6. **Construct and execute the request**
   Use `curl` to call the API.
   - **Base URL**: use `https://open-api.coinank.com` unless the schema specifies a different server.
   - **Authentication**: send the API key in the request header as `apikey: $COINANK_API_KEY`.
   - **Timestamps**: if the endpoint accepts `endTime`, prefer a current millisecond timestamp unless the user explicitly requested another time.
   - **Examples**: timestamps shown in the OpenAPI files are examples only and must not be reused as-is.


## Critical Rules

- **Do not bulk-load schemas**
  Unless the user explicitly asks for a cross-category analysis, do not open multiple OpenAPI JSON files at once.

- **Do not invent parameters**
  Pass only the parameters defined by the selected OpenAPI schema. Some endpoints return empty results when extra parameters are added.

- **Validate required arguments first**
  Before making a request, ensure all required parameters are present and conform to the schema.

- **Handle failures clearly**
  If the API request fails, explain the failure in user-friendly language and preserve the technical error details for troubleshooting.

- **Respect the documented response shape**
  The success indicator is `"code": "1"` as a string, not a number.


## API Key Setup

Users must provide their own API key through the environment:

```bash
export COINANK_API_KEY="your_api_key"
```


## Timestamp Rules

### `endTime` must be a current millisecond timestamp

```bash
# Correct
NOW=$(python3 -c "import time; print(int(time.time()*1000))")

# Wrong on macOS: %3N is not supported
NOW=$(date +%s%3N)
```

If an endpoint requires `endTime`, use a current millisecond timestamp unless the user explicitly specifies another valid time range.


## Parameter Rules

### Do not send unsupported parameters

Some endpoints do not accept `endTime` or `size`. For example, liquidation heatmap endpoints such as `getLiqHeatMap` can return empty data when unsupported parameters are included. Follow the selected schema exactly.

### `exchanges` is required for aggregate endpoints

For aggregate market-order endpoints such as `getAggCvd`, `getAggBuySellCount`, `getAggBuySellValue`, and `getAggBuySellVolume`, the `exchanges` parameter must be present. Use `exchanges=` to aggregate across all exchanges.

### `interval` values vary by endpoint

Supported `interval` values differ by API family. Always confirm the allowed values in the selected schema's parameter descriptions.

| Endpoint Type | Supported `interval` Values |
|---------|------------|
| K-line / market-order stats / long-short ratio / open interest | `1m, 3m, 5m, 15m, 30m, 1h, 2h, 4h, 6h, 8h, 12h, 1d` |
| Liquidation heatmap (`getLiqHeatMap`) | `12h, 1d, 3d, 1w, 2w, 1M, 3M, 6M, 1Y` |
| RSI screener | `1H, 4H, 1D` |
| Funding-rate heatmap | `1D, 1W, 1M, 6M` |


## Response Handling

Successful responses use `"code": "1"`. Some endpoints return nested payloads inside `data`, for example:

```json
{"success": true, "code": "1", "data": {"success": true, "code": "1", "data": [...]}}
```

Always inspect the actual response shape and unwrap nested `data` fields when necessary.


## Notes on OpenAPI Examples

Values shown in `references/*.json`, especially timestamps in `example` fields, are historical examples only. Replace them with live values when building requests.
