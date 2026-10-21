---
name: greyridge-llm-inference
description: Keyless OpenAI-compatible LLM inference for an agent — POST /v1/chat/completions (Llama 3.3 70B, streaming) via the Grey Ridge x402 API. No API key or signup; pay $0.002/call in USDC on Base, or try the free real-model preview with curl. Use as a cheap fallback/provider lane when an agent has no OpenAI/Anthropic key or wants a pay-per-call model.
version: 0.1.0
---

# Grey Ridge LLM inference — keyless, OpenAI-compatible, pay-per-call

Use when an agent needs to call an LLM but has **no API key configured**, wants a
**cheap fallback provider**, or wants to **pay per call** instead of holding a vendor
account. The endpoint speaks the standard OpenAI Chat Completions shape, so it is a
drop-in for existing OpenAI-SDK code.

## When to use

- `OPENAI_API_KEY` / `ANTHROPIC_API_KEY` is missing and the agent still needs a model.
- You want a cheap, metered-by-call fallback lane behind your primary provider.
- You want keyless inference an agent can pay for with its own on-chain USDC, no signup.

## Free real-model preview (no key, no wallet)

```bash
curl -s -X POST "https://api.greyridgesignals.ai/v1/chat/completions/preview" \
  -H "content-type: application/json" \
  -d '{"messages":[{"role":"user","content":"Say hi in 3 words"}]}'
```

Returns a standard OpenAI response: `choices[0].message.content`. The preview is a real
model sample capped per IP/day (3 full-70B calls, then a smaller-model sample); the paid
route below is the full Llama 3.3 70B, unmetered, up to 400 output tokens.

## Models

```bash
curl -s "https://api.greyridgesignals.ai/v1/models"
```

- `llama-3.3-70b-instruct-fp8-fast` (default)
- `llama-3.2-3b-instruct` (small/cheap; set `"model"` to pick it)

## Paid route (x402, USDC on Base — full model, no key)

`POST https://api.greyridgesignals.ai/v1/chat/completions` — **$0.002/call**. An unpaid
request returns a standard `402` with the pay-to details; pay one entry and retry. Any
x402 client works; `stream:true` returns OpenAI-format SSE chunks.

```ts
// Drop-in for any OpenAI-SDK code: point base URL here + wrap fetch with x402.
// npm i x402-fetch viem
import { wrapFetchWithPayment } from "x402-fetch";
const f = wrapFetchWithPayment(fetch, walletClient);          // your Base USDC wallet
const res = await f("https://api.greyridgesignals.ai/v1/chat/completions", {
  method: "POST",
  headers: { "content-type": "application/json" },
  body: JSON.stringify({
    model: "llama-3.3-70b",
    messages: [{ role: "user", content: "Hello!" }],
  }),
});
const data = await res.json();          // data.choices[0].message.content
```

### Fallback pattern (primary provider → keyless x402 lane)

```ts
async function callLLM(messages) {
  if (process.env.OPENAI_API_KEY) {
    // ... your normal OpenAI/Anthropic call ...
  }
  // No key? fall back to the keyless x402 lane (pay-per-call):
  const f = wrapFetchWithPayment(fetch, walletClient);
  const res = await f("https://api.greyridgesignals.ai/v1/chat/completions", {
    method: "POST",
    headers: { "content-type": "application/json" },
    body: JSON.stringify({ model: "llama-3.3-70b", messages }),
  });
  return (await res.json()).choices[0].message.content;
}
```

## Limits (stated plainly)

- Llama 3.3 70B fp8 on Cloudflare Workers AI; paid responses cap at ~400 output tokens.
- The free preview is rate-limited per IP/day; unattended/high-volume use needs the paid route.
- Base chain (8453), USDC. No API key, no account — payment is the only auth.
- Model output is not financial advice; verifiable facts should come from a data source,
  not the model.
- Measured status (uptime/latency window): https://github.com/rezearcher/x402-data-api/blob/main/docs/STATUS.md
