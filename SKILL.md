---
name: coinank-openapi
description: Access CoinAnk OpenAPI through direct API-key auth or pay via Agent Payments Protocol or x402
metadata:
  {
    "openclaw":
      {
        "homepage": "https://coinank.com",
        "priority": 100,
        "keywords": ["bitcoin", "btc", "ethereum", "eth", "cryptocurrency", "crypto", "price", "trend", "liquidation", "long-short ratio", "agent payments protocol", "x402", "pay-per-call", "payment"]
      },
  }
---

# Permissions
# SECURITY MANIFEST:
# - Allowed to read: {baseDir}/README.md, {baseDir}/references/*.json
# - Allowed to make network requests to: https://open-api.coinank.com


## Access Modes

This skill supports two access paths:

1. **Direct mode** — use `COINANK_API_KEY` in the `apikey` request header.
2. **Pay-per-call mode** — pay via Agent Payments Protocol or x402 when CoinAnk returns an HTTP `402 Payment Required` challenge, then replay the same request.

`COINANK_API_KEY` is optional. If it is not present, the skill must still attempt access discovery and use Agent Payments Protocol or x402 when available.

## First-Time User Guidance

When a new user starts using this skill, make the access options explicit:

- If the user already has a CoinAnk API membership, tell them to provide `COINANK_API_KEY`.
- If the user does not have a CoinAnk API membership, tell them they can still try Agent Payments Protocol or x402 pay-per-call access when CoinAnk returns a payment challenge.
- Do not present API membership as the only way to use the skill.


## Dependencies for Agent Payments Protocol / x402

Agent Payments Protocol / x402 pay-per-call mode depends on the OKX Onchain OS payment stack:

- `okx-x402-payment`
- `okx-agentic-wallet`

If an Agent Payments Protocol or x402 flow is needed but those skills are unavailable, instruct the user to install `okx/onchainos-skills` first.


## Agent Payments Protocol / x402 Signing Scheme Constraint

When a payment challenge is signed, the signing scheme must match the signer type:

- **EOA private key** → use the **`exact`** scheme.
- **OKX contract wallet / OKX wallet session signing** → use the **`aggr_deferred`** scheme.

Do not mix these paths:

- Do not use `aggr_deferred` for EOA private-key signing.
- Do not use `exact` for OKX contract-wallet signing.


## Operating Mode

This skill must operate in an on-demand loading mode. Do not read every OpenAPI file by default. Load only the schema needed for the user's request.


## Required Workflow

When handling a user request, follow this sequence strictly:

1. **Read the project README**
   Read `README.md` before making any request so you follow the documented conventions, access modes, and edge cases.

2. **Identify the relevant API category**
   Scan the filenames under `{baseDir}/references/` and determine which OpenAPI file matches the user's request.

3. **Read only the required schema**
   Open only the selected `.json` file and inspect its `paths`, parameters, response shape, and endpoint-specific restrictions. In the `paths` object, each key is an API path.

4. **Validate request parameters**
   Confirm required parameters, supported enum values, and whether the endpoint accepts optional fields such as `endTime`, `size`, `interval`, or `exchanges`.

5. **Choose the access strategy**
   - If `COINANK_API_KEY` is present, prefer **direct mode** first.
   - If `COINANK_API_KEY` is absent, use **discovery mode**: send the original request without an API key and let the server tell you whether the route is public, payable via Agent Payments Protocol or x402, or still unavailable without membership.

6. **Construct the original request**
   Build the exact request the user asked for.
   - **Base URL**: use `https://open-api.coinank.com` unless the schema specifies a different server.
   - **Authentication**:
     - Direct mode: send `apikey: $COINANK_API_KEY`
     - Discovery mode: do not send `apikey` and do not fabricate any payment header before seeing a real `402`
   - **Timestamps**: if the endpoint accepts `endTime`, prefer a current millisecond timestamp unless the user explicitly requested another time.
   - **Examples**: timestamps shown in OpenAPI example fields are historical examples only and must not be reused as-is.

7. **Send the original request first**
   Always send the original request before attempting wallet login, payment signing, or payment header construction.

8. **Interpret the response**
   - **HTTP 2xx + business code `"1"`**: return the result.
   - **HTTP 402**: this route is payment-gated. Switch into the Agent Payments Protocol / x402 payment flow.
   - **HTTP 2xx + business code `"-3"`**:
     - If an API key was supplied, treat it as invalid or insufficient and tell the user to fix their CoinAnk access.
     - If no API key was supplied and no HTTP 402 challenge was returned, explain that the request did not enter the Agent Payments Protocol / x402 payment path and still cannot be completed without valid access.
   - **Other failures**: explain the failure clearly and include the key technical reason.

9. **Run the Agent Payments Protocol / x402 payment flow only after a real HTTP 402**
   Use `okx-x402-payment` and follow its confirmation, login, signing, and replay flow.
   - Do not check wallet status before receiving HTTP 402.
   - Do not log in preemptively.
   - Do not charge speculatively.
   - If signing with an **EOA private key**, use the **`exact`** scheme.
   - If signing with an **OKX contract wallet / wallet session**, use the **`aggr_deferred`** scheme.

10. **Replay the exact same request**
   After payment proof is generated, replay the same method, URL, query parameters, and request body. Only add the payment header required by the Agent Payments Protocol / x402 flow.

11. **Return the final paid response**
   Return the successful result from the replayed request, not the intermediate 402 payload.


## Multi-Call Guard

If the user asks for a wide analysis that would likely require multiple paid API calls and there is no valid `COINANK_API_KEY`, stop and warn that the task may incur multiple Agent Payments Protocol / x402 payments. Ask for confirmation before triggering a multi-call paid workflow.


## Critical Rules

- **Do not bulk-load schemas**
  Unless the user explicitly requests cross-category analysis, do not open multiple OpenAPI JSON files at once.

- **Do not invent parameters**
  Pass only the parameters defined by the selected schema. Some endpoints return empty results when extra parameters are added.

- **Do not invent payment support**
  Treat a request as Agent Payments Protocol / x402 payable only when CoinAnk actually returns an HTTP `402 Payment Required` challenge.

- **Do not mix signing schemes**
  Use `exact` for EOA private-key signing, and use `aggr_deferred` for OKX contract-wallet or OKX wallet-session signing.

- **Do not bypass the challenge**
  Never attempt Agent Payments Protocol / x402 signing unless you have already received a real 402 response for the exact request being made.

- **Do not mutate the paid request**
  The replayed request must match the original request exactly except for the payment header.

- **Validate required arguments first**
  Ensure all required parameters are present and schema-compliant before making the request.

- **Respect the documented response shape**
  CoinAnk success is `"code": "1"` as a string, not a number.

- **Handle failures clearly**
  If the request or payment flow fails, explain the issue in user-friendly language and preserve the technical cause for troubleshooting.


## API Key Mode

Users with CoinAnk membership can configure direct access:

```bash
export COINANK_API_KEY="your_api_key"
```

Use direct mode whenever a valid API key is available, unless the user explicitly asks to use Agent Payments Protocol or x402 pay-per-call payment instead.


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
|---|---|
| K-line / market-order stats / long-short ratio / open interest | `1m, 3m, 5m, 15m, 30m, 1h, 2h, 4h, 6h, 8h, 12h, 1d` |
| Liquidation heatmap (`getLiqHeatMap`) | `12h, 1d, 3d, 1w, 2w, 1M, 3M, 6M, 1Y` |
| RSI screener | `1H, 4H, 1D` |
| Funding-rate heatmap | `1D, 1W, 1M, 6M` |


## Response Handling

Successful CoinAnk responses use `"code": "1"`. Some endpoints return nested payloads inside `data`, for example:

```json
{"success": true, "code": "1", "data": {"success": true, "code": "1", "data": [...]}}
```

Inspect the actual response shape and unwrap nested `data` fields when necessary.


## Notes on Agent Payments Protocol / x402 Availability

CoinAnk supports Agent Payments Protocol or x402 pay-per-call access. In practice, the skill must still rely on the server's real HTTP behavior for each request. If a request does not return an HTTP 402 challenge, the skill must not fabricate a payment flow.


## Notes on OpenAPI Examples

Values shown in `references/*.json`, especially timestamps in `example` fields, are historical examples only. Replace them with live values when building requests.
