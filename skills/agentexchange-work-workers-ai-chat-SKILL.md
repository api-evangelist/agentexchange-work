# Workers AI chat (OpenAI-compatible)

POST https://store.agentexchange.work/v1/chat/completions
Price: $0.002 USDC on Base via x402. Unpaid request → HTTP 402 with payment requirements. Pay, retry with X-PAYMENT.
Body: {"messages":[{"role":"user","content":"hi"}]}
Response: OpenAI chat.completion { id, object, choices, model, usage }.
Free discovery: GET https://store.agentexchange.work/v1/models
Catalog: GET https://store.agentexchange.work/.well-known/x402
Alias: POST https://store.agentexchange.work/api/v1/chat/completions
