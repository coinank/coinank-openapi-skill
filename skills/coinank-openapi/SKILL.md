---
name: coinank-openapi
version: 1.0.0
description: Access CoinAnk crypto derivatives market data via API key or Agent Payments Protocol/x402 pay-per-call.
author: CoinAnk
tags:
  - bitcoin
  - ethereum
  - cryptocurrency
  - derivatives
  - market-data
  - liquidation
  - analytics
  - agent-payments-protocol
  - x402
  - payment
---

## Overview

CoinAnk OpenAPI provides access to cryptocurrency derivatives market data, including K-lines, ETFs, open interest, long/short ratios, funding rates, liquidations, order flow, whale activity, and related analytics. Use direct API-key authentication when available, or use Agent Payments Protocol / x402 pay-per-call access through the latest OKX payment skill when CoinAnk returns a payment challenge.

## Pre-flight Checks

Before using this skill, ensure:

1. Review the user's requested market-data task and select the matching reference OpenAPI file under `references/` when endpoint details are needed.
2. If `COINANK_API_KEY` is available, use API-key mode and send it only in the `apikey` request header.
3. If no API key is available, send the original request first and only start Agent Payments Protocol / x402 payment handling after a real HTTP `402 Payment Required` challenge or an OKX payment-skill `charge` requirement.
4. For paid challenges, use `okx-agent-payments-protocol`; do not implement wallet signing manually in this skill.
5. Confirm any payment with amount greater than zero with the user before payment execution. Zero-amount challenges are valid and must remain exactly zero.

## Commands

This is a skill-only plugin. The command names below describe agent workflows, not a shipped local binary.

### coinank-openapi quickstart

Use when the user is new to the plugin or has not chosen an access mode. Explain API-key access, Agent Payments Protocol / x402 pay-per-call access, and the OKX `charge` payment method when required by the OKX payment skill.

**Output**: Recommended access mode, required prerequisites, and the safest next request.

**Example**: Ask the agent to run the `coinank-openapi quickstart` flow before the first CoinAnk data request.

### coinank-openapi query-market-data

Use when the user asks for CoinAnk market data such as K-lines, funding rates, liquidations, open interest, long/short ratios, order flow, ETF data, whale activity, or trending symbols.

**Output**: A concise answer based on the selected CoinAnk endpoint, including key parameters used and any access or payment requirements encountered.

**Example**: Ask for BTC funding rates, ETH liquidation heatmap data, or Binance BTCUSDT open interest.

### coinank-openapi use-api-key

Use when `COINANK_API_KEY` is available or the user wants direct membership access. Send the request with the `apikey` header and do not start a pay-per-call flow unless the user explicitly requests it.

**Output**: Final API response summary or a clear authentication/API-level error.

**Example**: Query `/api/fundingRate/current` with the user's CoinAnk API key.

### coinank-openapi pay-per-call

Use only after CoinAnk returns a real HTTP `402 Payment Required` challenge or the OKX payment skill indicates a required payment flow. Delegate payment execution to `okx-agent-payments-protocol`, including x402 proof generation or the newer `charge` method when required.

**Output**: User-facing payment summary, confirmation request when amount is greater than zero, and final replayed API response after successful payment.

**Example**: Complete a zero-amount payment challenge without coercing `0` to `0.000001`, then replay the same request with the generated payment data.

## Access Modes

This skill supports two access paths:

1. **Direct mode** -- use `COINANK_API_KEY` in the `apikey` request header.
2. **Pay-per-call mode** -- pay via Agent Payments Protocol or x402 when CoinAnk returns an HTTP `402 Payment Required` challenge, then replay the same request.

`COINANK_API_KEY` is optional. If it is not present, the skill must still attempt access discovery and use Agent Payments Protocol or x402 when available.

## First-Time User Guidance

When a new user starts using this skill, make the access options explicit:

- If the user already has a CoinAnk API membership, tell them to provide `COINANK_API_KEY`.
- If the user does not have a CoinAnk API membership, tell them they can still try Agent Payments Protocol or x402 pay-per-call access when CoinAnk returns a payment challenge.
- Do not present API membership as the only way to use the skill.


## Dependencies for Agent Payments Protocol / x402

Agent Payments Protocol / x402 pay-per-call mode depends on the OKX Onchain OS payment stack:

- `okx-agent-payments-protocol`
- `okx-agentic-wallet`

Use the latest `okx-agent-payments-protocol` skill. It supports both payment-proof generation for Agent Payments Protocol / x402 challenges and the newer `charge` payment method. Do not hard-code an older x402-only flow; delegate payment execution to `okx-agent-payments-protocol` and follow the payment method returned or required by that skill.

If an Agent Payments Protocol, x402, or `charge` payment flow is needed but those skills are unavailable, instruct the user to install or update `okx/onchainos-skills` first.


## Zero-Amount Payment Challenges

Agent Payments Protocol / x402 challenges with an amount of `0` are valid and must be supported. Do not treat a zero-amount challenge as malformed, unpaid, or unsupported.

When the challenge amount is `0`:

- Clearly tell the user that the request requires a payment proof but the charge amount is zero.
- Continue through `okx-agent-payments-protocol` to generate the required proof or authorization with the original zero amount.
- Replay the exact same request with the generated payment header.
- Do not replace `0` with any fallback, minimum, dust, or micro amount such as `0.000001` USDC/USDT.
- Do not require paid-call confirmation solely because the amount is zero, but still respect any wallet or protocol confirmation required to sign/authorize the proof.


## Agent Payments Protocol / x402 Signing Scheme Constraint

When a payment challenge is signed, the signing scheme must match the signer type:

- **EOA private key** -> use the **`exact`** scheme.
- **OKX contract wallet / OKX wallet session signing** -> use the **`aggr_deferred`** scheme.

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

9. **Run the Agent Payments Protocol / x402 payment flow only after a real payment requirement**
   Use `okx-agent-payments-protocol` and follow its confirmation, login, signing, charge, and replay flow.
   - For HTTP `402 Payment Required` challenges, generate the required Agent Payments Protocol / x402 payment proof and replay the same request.
   - If the latest OKX payment skill indicates or requires the newer `charge` payment method, use that `charge` flow instead of forcing the legacy x402-only proof path.
   - Do not check wallet status before a real payment requirement is known.
   - Do not log in preemptively.
   - Do not charge speculatively.
   - If the challenge amount is `0`, treat it as a valid zero-amount payment challenge and still generate the required proof or authorization using exactly `0`; never coerce it to `0.000001` or any other minimum non-zero amount.
   - If signing with an **EOA private key**, use the **`exact`** scheme when the OKX payment flow uses x402 proof signing.
   - If signing with an **OKX contract wallet / wallet session**, use the **`aggr_deferred`** scheme when the OKX payment flow uses x402 proof signing.

10. **Replay the exact same request**
   After the OKX payment flow completes, replay the same method, URL, query parameters, and request body. Only add the payment header or authorization data required by the selected Agent Payments Protocol / x402 or `charge` flow.

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
  Treat a request as Agent Payments Protocol / x402 payable only when CoinAnk actually returns an HTTP `402 Payment Required` challenge. A challenge amount of `0` is still a valid payment challenge and must remain exactly zero throughout proof generation.

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
## Error Handling

| Error | Cause | Resolution |
|-------|-------|------------|
| HTTP `401` | Missing, invalid, or rejected `apikey`, or payment data rejected by the gateway/upstream service | Check whether API-key mode or pay-per-call mode is intended. Do not retry with fabricated credentials. If using pay-per-call, request a fresh challenge and replay the same request with the generated payment data. |
| HTTP `402 Payment Required` | CoinAnk requires Agent Payments Protocol / x402 payment for this request | Delegate payment handling to `okx-agent-payments-protocol`. Confirm non-zero amounts with the user, support zero-amount challenges exactly as returned, and replay the original request after payment proof or `charge` completion. |
| Missing or unsupported endpoint parameters | Required query parameters are absent, stale, or not accepted by the selected CoinAnk endpoint | Read the relevant OpenAPI reference file, ask the user for missing values, and follow timestamp and symbol formatting rules. |
| Payment-skill unavailable | The required OKX payment helper is not installed or not accessible | Explain that pay-per-call mode requires `okx-agent-payments-protocol`; ask the user to install or enable it, or use API-key mode if available. |
| Rate limit, timeout, or upstream unavailable | CoinAnk or a CDN/cache layer temporarily rejected or failed the request | Report the failure clearly, avoid duplicate paid replays without user confirmation, and retry only when safe. |

## Security Notices

- This plugin is for market-data retrieval and payment orchestration only; it does not execute trades or move assets by itself.
- Never request, print, persist, or expose private keys, seed phrases, API keys, payment proofs, authorization headers, or wallet-session credentials.
- Use `COINANK_API_KEY` only from the user's configured environment or secret store, and send it only as the CoinAnk `apikey` header.
- Delegate Agent Payments Protocol / x402 proof generation, OKX wallet signing, and `charge` handling to `okx-agent-payments-protocol`; do not bypass its confirmation flow.
- Confirm every non-zero payment amount with the user before execution. Do not charge speculatively.
- Treat zero-amount payment challenges as valid, but never coerce zero to a non-zero fallback amount.

