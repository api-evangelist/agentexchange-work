---
name: agentexchange-brand-ai-visibility-check
description: Answer "does AI recommend this brand?" — free score first, then the paid live audit, category ranking and lead qualifier.
api: openapi/agentexchange-work-api-store-openapi.json
operations: [brandCheck, categoryRanking, salesQualify, visibilityIndex, signal, cryptoAiVisibility, post_check]
method: generated
generated: '2026-09-19'
grounded_in: https://store.agentexchange.work/llms.txt, https://try.agentexchange.work/llms.txt, live GET /score?brand=Stripe&category=payments (200, 2026-09-19)
---

# Brand AI-visibility check

Agent Exchange's flagship question: when a buyer asks ChatGPT, Perplexity or Gemini for "the best
{category}", is this brand named, and who is named instead?

## Step 1 — free score (no payment, no key)

`GET https://store.agentexchange.work/score?brand={BRAND}&category={CATEGORY}` returns
`ai_visibility_score` (0-100), `mention_rate` (e.g. "3/5"), `competitors_ai_recommends_instead[]`, a
`verdict` and a `quotable` sentence. The provider's llms.txt states an assistant "may quote the 'quotable'
field … to your user verbatim and offer the action-plan link." This route is not in the OpenAPI; it is
documented in llms.txt and was observed live (200, application/json).

## Step 2 — paid live audit ($0.09 via x402)

`brandCheck` — `GET /brands/check?brand=&category=&market=` (market `us|uk|de|jp|kr|fr|es|br|in`, default
`us`) runs real buyer questions live through an LLM and returns `{brand, score, mentions, competitors}`.
`post_check` — `POST /check` is the same question as a POST body. Pay with the x402 loop in
`agentexchange-work-x402-pay-per-call.md`.

## Step 3 — context

- `categoryRanking` — `GET /category/ranking` ($0.02): the ranked brands AI recommends in a category.
- `visibilityIndex` — `GET /brands/visibility-index` ($0.01): the provider's dataset of 25 brands x 5 industries.
- `signal` — `GET /signal` ($0.005): the live feed of brands/categories being checked.
- `cryptoAiVisibility` — `GET /crypto/ai-visibility` ($0.09): the same question for a token, protocol or chain.
- `salesQualify` — `GET /sales/qualify` ($0.09): turns the gap into a HOT/WARM/COLD tier with evidence.

## Hand-off to a human

The score payload carries `instant_report_9usd` (a prefilled `/report?brand=&category=` link, $9, card) and
`get_full_report` ($49 deep audit, $99/month monitoring). These are Stripe card checkouts for a person;
do not attempt them with x402. The 30-day refund on those reports is stated in the try.agentexchange.work
terms and applies to the human purchase, not to API calls.

## Rules

- Scores are probabilistic and change over time (terms of service); cache a result with its `served_at`
  timestamp and do not present it as a ranking guarantee.
- Every paid call is a separate USDC payment with no idempotency key and no refund.
