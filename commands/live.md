---
description: Ask Grok with Live Search enabled — real-time facts from X (Twitter), web, and news, with citations. For breaking developments and "what's being said right now" where X is the primary source.
argument-hint: <question> [--sources x,web,news]
---

Forward $ARGUMENTS to xAI Grok via the `grok-rescue` subagent with **Live Search enabled**, and return the grounded answer plus citation URLs.

Grok's Live Search pulls current data from X (Twitter), the web, and news. This is its distinct edge over other research tools: **real-time X access**.

Use for:

- "What is being said on X right now about <topic>"
- Breaking news / developments after the training cutoff
- Sentiment or reaction on X to an announcement
- Any fact where X is the primary, fastest source

## Sources

Tools: `web_search` (web) and `x_search` (X/Twitter). Use both by default; pass `--sources x` to use `x_search` only, `--sources web` for web only.

## Request shape (Agent Tools API)

The legacy `search_parameters` field is deprecated. Search now runs as server-side **tools** on the `/v1/responses` endpoint:

```bash
curl -s "https://api.x.ai/v1/responses" \
  -H "Authorization: Bearer $XAI_API_KEY" -H 'Content-Type: application/json' \
  -d '{"model":"grok-4.3","stream":false,
       "input":[{"role":"user","content":"<question>"}],
       "tools":[{"type":"web_search"},{"type":"x_search"}]}'
```

The answer is in `output[].content[].text`; citation URLs are in `output[].content[].annotations[].url`. Return the answer, then a `Sources:` list. Search adds a per-call cost — keep queries focused.

If $ARGUMENTS is empty, ask the user what to search.
