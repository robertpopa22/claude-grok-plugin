---
name: grok-api-runtime
description: Internal helper contract for calling the xAI Grok API from Claude Code. Defines auth, the OpenAI-compatible endpoint, model selection, Live Search, request/response shapes, citation handling, and error handling.
---

# xAI Grok API Runtime Contract

This skill documents how the `grok-rescue` subagent calls the xAI Grok API. The API is **OpenAI-compatible** — the runtime is plain `curl` against `https://api.x.ai/v1` (no CLI dependency).

## Prerequisites

1. **API key**: a paid xAI API key. Stored in Vault `xAI-Grok-API` (field `apikey`), and exposed to the shell as the `XAI_API_KEY` env var (persisted via `setx` on Windows / shell profile on *nix).
2. **No package install** — `curl` is available; `python` only for JSON parsing.

## Auth

Every request carries `Authorization: Bearer $XAI_API_KEY`. Never inline the literal key in committed files — read it from the env var (or Vault for recovery).

## Endpoint

OpenAI-compatible chat completions:

```bash
curl -s "https://api.x.ai/v1/chat/completions" \
  -H "Authorization: Bearer $XAI_API_KEY" -H 'Content-Type: application/json' \
  -d '{"model":"grok-4.3","messages":[{"role":"user","content":"<prompt>"}]}'
```

- Answer at `.choices[0].message.content`.
- List models: `GET https://api.x.ai/v1/models`. Rich metadata (context, pricing, modalities): `GET https://api.x.ai/v1/language-models`.

## Model slugs (live-probed 2026-06-10)

| Slug | Status | Notes |
|------|--------|-------|
| `grok-4.3` | ✅ OK | **Default** — flagship. Aliases: `grok-latest`, `grok-4.3-latest`, `grok-3`. text + image input, >200K context. |
| `grok-4.20-0309-reasoning` | ✅ OK | Explicit step-by-step reasoning variant. |
| `grok-build-0.1` | ✅ OK | Code-focused agent model. |
| `grok-4.20-0309-non-reasoning` | ✅ (list) | Non-reasoning fast variant. |
| `grok-4.20-multi-agent-0309` | ⚠ | Returns a non-standard response shape — avoid as default. |
| `grok-imagine-*` | n/a | Image/video generation — not for text rescue. |

**Recommended default**: `grok-4.3`. Use `grok-4.20-0309-reasoning` for step-by-step reasoning, `grok-build-0.1` for code.

> **Context tiers**: `grok-4.3` has `long_context_threshold: 200000` — prompts above ~200K tokens bill at the higher long-context rate. Keep prompts under 200K when possible.

> **Re-verify cadence**: re-probe slugs after any xAI model announcement.
>
> ```bash
> curl -s "https://api.x.ai/v1/models" -H "Authorization: Bearer $XAI_API_KEY" \
>   | python -c "import sys,json;[print(m['id']) for m in json.load(sys.stdin).get('data',[])]"
> ```

## Live Search via Agent Tools API (the distinct edge)

Grok can pull **real-time data from X (Twitter) and the web** with citations. The legacy `search_parameters` field on `/chat/completions` is **deprecated** (the API returns: *"Live search is deprecated. Please switch to the Agent Tools API"*).

Server-side search now runs as **tools** on the `/v1/responses` endpoint (verified working 2026-06-10):

```bash
curl -s "https://api.x.ai/v1/responses" \
  -H "Authorization: Bearer $XAI_API_KEY" -H 'Content-Type: application/json' \
  -d '{"model":"grok-4.3","stream":false,
       "input":[{"role":"user","content":"<question>"}],
       "tools":[{"type":"web_search"},{"type":"x_search"}]}'
```

- `input` (not `messages`) is the message array on `/v1/responses`.
- `tools`: `{"type":"web_search"}` and/or `{"type":"x_search"}`. Omit a tool to disable that source.
- `stream:false` to get one parseable JSON object (default is streaming).
- **Search runs server-side and adds a per-call cost** — keep queries focused, do not enable by default.

Response shape: `output[]` is a list of steps (reasoning + tool calls + final `message`). The answer text is at the `message` item's `content[].text`; citation URLs are at that content's `annotations[].url`:

```bash
... | python -c "
import sys,json
d=json.load(sys.stdin)
for o in d.get('output',[]):
    if o.get('type')=='message':
        for c in o.get('content',[]):
            print(c.get('text',''))
            ann=c.get('annotations',[])
            if ann: print('\nSources:'); [print('-',a.get('url','')) for a in ann]
"
```

## Error handling

- **401 / `Unauthorized`**: `XAI_API_KEY` missing/wrong in the shell. Confirm it is exported (key in Vault `xAI-Grok-API`), then retry. Do NOT loop.
- **404 model**: slug retired or wrong — re-list with `/v1/models`.
- **429 / quota**: report verbatim; paid plan has rate/credit limits. Do not silently downgrade.
- **Network error**: report verbatim, do not auto-retry.

## Anti-patterns

- **Do NOT** inline the API key in any committed file — env var only.
- **Do NOT** enable Live Search by default — it bills per search. Only for time-sensitive / X questions.
- **Do NOT** use `grok-imagine-*` (image/video) for text tasks.
- **Do NOT** feed Claude's conclusion as fact in an adversarial review — instruct Grok to re-derive the premise.

## Cost

Pay-per-token (see `/v1/language-models` for current rates; long-context tier applies above 200K tokens). Live Search adds a per-search fee. Set a spend alert in the xAI console. A handful of second-opinion calls/day is a few dollars at most.
