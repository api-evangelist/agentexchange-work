---
name: agentexchange-x402-pay-per-call
description: Call any paid Agent Exchange API Store route — read the 402, pay in USDC over x402, retry — using the on-chain read routes as the worked example.
api: openapi/agentexchange-work-api-store-openapi.json
operations: [chainHeartbeat, chainBalance, chainGas, chainTx, chainConfirmations, chainId, blockNumber]
related:
  api: openapi/agentexchange-work-gatekeeper-oracle-openapi.json
  operations: [oracleInfo]
method: generated
generated: '2026-09-19'
grounded_in: https://store.agentexchange.work/api-docs, https://store.agentexchange.work/llms.txt, live 402 on GET /chain/gas (2026-09-19)
---

# Pay per call on the Agent Exchange API Store

Every paid route on `https://store.agentexchange.work` is keyless. There is no account, no API key and no
OAuth; the payment receipt is the credential. Prices are quoted in the provider's canonical catalog,
`GET https://store.agentexchange.work/.well-known/x402`, which the provider says is "authoritative for live
prices" — read it before you budget, not the OpenAPI, if the two ever disagree.

## Before you pay

1. `GET /samples` (free) returns an illustrative snapshot of every paid product with the exact upgrade call.
   The provider labels these "illustrative snapshots" — do not treat them as live data.
2. `GET /status` and `GET /health` (free) confirm the store is operational and report `paid_routes`.
3. For chain work, `chainHeartbeat` (`GET /chain/heartbeat`, $0.09) is the provider's one pre-action read:
   chain identity, block height and freshness in one payment. If you only need one fact, the cheapest reads
   are `chainId` ($0.001), `blockNumber` ($0.001), `chainBalance` ($0.003) and `chainTx` ($0.008).

## The loop (x402)

1. Send the request with no payment, e.g. `GET /chain/gas?chain=base` (`chainGas`).
2. You receive **HTTP 402**. The body is an x402 v1 envelope: `accepts[]` carries `maxAmountRequired` in
   atomic USDC (6 decimals — `"20000"` is $0.02), `payTo`, `asset`, `network` (`base` or `solana`) and
   `maxTimeoutSeconds` (300). The same requirements arrive base64-encoded in the `PAYMENT-REQUIRED` header
   and in `WWW-Authenticate: MPP …` (x402 v2 shape). A `Link: </ask>; rel="alternate"` header offers a card
   checkout for humans — never try to pay that with x402.
3. Sign a USDC authorization for exactly `maxAmountRequired` on the rail you chose: EIP-3009
   `transferWithAuthorization` on Base (`eip155:8453`, USDC `0x8335…2913`) or an SPL transfer on Solana.
4. Retry the **identical** request with the base64 payload in `X-PAYMENT` (x402 v1) or `PAYMENT-SIGNATURE`
   (x402 v2). You get the data with a 200.

Over MCP (`https://store.agentexchange.work/mcp`) the same loop runs inside `tools/call`: the first call
returns the requirements; retry with the signed payload in the `x_payment` argument.

## Rules an agent must follow here

- **No idempotency key exists.** A retried paid request is a second payment. Do not blind-retry a paid call
  after a timeout; check `chainTx` / `chainConfirmations` on your own authorization first.
- **Nothing is reversible.** There is no cancel, refund or undo route; a settled transfer is final. The only
  stated protection is on the Gatekeeper Oracle (`GET /oracle/info`, `oracleInfo`, free): "If every backing
  model fails the route returns 503 and settlement is cancelled."
- **Rate limits are unpublished and shared.** A burst across `*.agentexchange.work` draws a Cloudflare
  `429 text/plain "error code: 1015"` with no `Retry-After`; back off ~30 s.
- **Errors are not RFC 9457.** Expect `{error, path, message}` on 404 and the x402 envelope on 402.
- **Free first.** `GET /score?brand=&category=` is a free, keyless, CORS-enabled live call — use it before
  paying $0.09 for `brandCheck` if a score is all you need.
