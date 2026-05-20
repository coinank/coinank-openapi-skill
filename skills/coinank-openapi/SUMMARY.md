# coinank-openapi

## Overview

CoinAnk OpenAPI gives agents access to cryptocurrency derivatives market data across K-lines, ETFs, open interest, long/short ratios, funding rates, liquidations, order flow, whale activity, and related analytics.

The plugin supports direct CoinAnk API-key access and pay-per-call access through Agent Payments Protocol / x402 via the latest OKX payment skill, including the newer `charge` payment method when required by the OKX payment flow.

## Prerequisites

- Optional: a CoinAnk API key with the required API level for the requested endpoint.
- For pay-per-call access: install or update `okx/onchainos-skills` so `okx-agent-payments-protocol` and `okx-agentic-wallet` are available.
- Network access to `open-api.coinank.com`.

## Quick Start

1. Ask the agent to run the `coinank-openapi quickstart` flow to review access options and choose API-key mode or Agent Payments Protocol / x402 pay-per-call mode.
2. If you have a CoinAnk API key, set `COINANK_API_KEY` and call the desired endpoint with the `apikey` header.
3. If you do not have an API key, send the original request first and let CoinAnk return an HTTP `402 Payment Required` challenge when pay-per-call access is available.
4. Use the latest `okx-agent-payments-protocol` skill to complete the required proof, authorization, or `charge` flow, then replay the same request with the generated payment data.
