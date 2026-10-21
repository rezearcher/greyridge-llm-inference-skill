# greyridge-llm-inference-skill

A Hermes / agent skill: **keyless, OpenAI-compatible LLM inference** for any agent,
served by the [Grey Ridge x402 API](https://api.greyridgesignals.ai).

- **`POST /v1/chat/completions`** — Llama 3.3 70B (fp8), `stream:true` supported,
  standard OpenAI request/response shape (`choices[0].message.content`).
- **No API key, no account, no signup.** Pay per call with [x402](https://x402.org)
  in USDC on Base (`$0.002`/call). An unpaid call returns a standard `402` with the
  pay-to details, so any x402 client works unchanged.
- **Free real-model preview** with `curl` (no wallet): `POST /v1/chat/completions/preview`.

```bash
# free real-model preview (no key, no wallet)
curl -s -X POST "https://api.greyridgesignals.ai/v1/chat/completions/preview" \
  -H "content-type: application/json" \
  -d '{"messages":[{"role":"user","content":"Say hi in 3 words"}]}'
```

Point any OpenAI SDK's base URL at `https://api.greyridgesignals.ai/v1` with an x402
fetch wrapper for the full, unmetered paid route.

Skill definition: [`skills/greyridge-llm-inference/SKILL.md`](skills/greyridge-llm-inference/SKILL.md).
Companion skills:
[greyridge-market-data-skill](https://github.com/rezearcher/greyridge-market-data-skill) ·
[greyridge-token-safety-skill](https://github.com/rezearcher/greyridge-token-safety-skill).

MIT licensed. Model output is not financial advice.
