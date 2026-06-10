# claude-grok-plugin

Use **xAI Grok** from Claude Code for an **independent fourth-family second opinion** and **real-time Live Search** over X (Twitter), web, and news — delegated to a `grok-rescue` subagent.

This is the family-diversity + real-time member of a multi-model workflow: **Claude** drives, **Codex** does write-capable code rescue, **Gemini** gives long-context second opinions, **Perplexity** brings cited web research, and **Grok** adds a model family independent of all three plus live X access.

## What you get

- **`@grok-rescue` subagent** — thin forwarding wrapper, returns Grok's answer (with citations when Live Search is used)
- **`/grok:rescue <task>`** — delegate to Grok for a fourth-family second opinion or alternative reasoning (default `grok-4.3`)
- **`/grok:live <question>`** — answer with **Live Search** enabled: real-time facts from X, web, and news, with citations
- **API-key auth** — paid xAI key (`XAI_API_KEY` env var); OpenAI-compatible REST, no CLI

## Why Grok alongside Claude

Two reasons:

1. **Family diversity for adversarial review.** Grok (xAI) is independent of Anthropic, OpenAI, and Google. When three reviewers share a wrong premise they fail identically — a distinct family breaks that correlation. Grok is the diversity vote next to Gemini and Codex.
2. **Real-time X access.** Grok's Live Search reads X (Twitter), web, and news live, with citations. For "what's being said right now" or breaking developments where X is the primary source, that's an edge no other family offers.

The xAI API is **OpenAI-compatible**, so this plugin calls it directly with `curl` — no fragile community-CLI dependency.

## Install

### 1. Get an xAI API key

Create a key in the [xAI console](https://console.x.ai) (paid). Then export it:

```bash
# persist via setx on Windows / shell profile on *nix
export XAI_API_KEY="xai-..."
```

### 2. Install this plugin

```bash
git clone https://github.com/robertpopa22/claude-grok-plugin \
  ~/.claude/plugins/cache/geseidl-grok/grok/0.1.0
```

Restart Claude Code. The `@grok-rescue` agent and `/grok:*` commands will be available.

## Usage

### Fourth-family second opinion

```
/grok:rescue Claude concluded the WAN drops every 2.5 min — verify from the raw ping log independently and say if you agree.
```

### Real-time X / web (Live Search)

```
/grok:live Ce se discuta acum pe X despre lansarea Next.js 16?
```

Returns a grounded answer plus a `Sources:` list of citation URLs.

### Direct subagent use (from main thread)

> *Claude*: "Gemini and Codex agreed, but they shared my framing. Let me get an independent read from Grok before we commit."

## Configuration

| Setting | Default | How to change |
|---------|---------|---------------|
| Model | `grok-4.3` | Pass `-m grok-4.20-0309-reasoning` (reasoning) or `-m grok-build-0.1` (code) |
| Live Search | off | `/grok:live` turns it on (Agent Tools API: `web_search` + `x_search`) |
| Auth | API key | `XAI_API_KEY` env var |

Model slugs (probed 2026-06-10): `grok-4.3` (flagship, alias `grok-latest`), `grok-4.20-0309-reasoning`, `grok-build-0.1`. Context tier rises above 200K tokens.

## Versioning

**v0.1.0** — initial release. `/grok:rescue` + `/grok:live` + `grok-rescue` agent, OpenAI-compatible curl runtime, no background job queue.

## Maintained by

<a href="https://geseidl.ro/servicii-it"><img src="https://geseidl.ro/assets/icons/logo-green.png" alt="Geseidl Consulting Group" height="40"></a>

This plugin is part of our **multi-model workflow** at Geseidl IT Solutions — Claude for primary development, Codex for write-capable rescue, Gemini for second-opinion review, Perplexity for cited web research, and Grok for an independent fourth-family read plus real-time X. Open-sourcing the integration because great tools should be shared.

Maintained by [Geseidl IT Solutions](https://geseidl.ro/servicii-it), part of [Geseidl Consulting Group](https://geseidl.ro).

## License

MIT — see [LICENSE](LICENSE).

---

<p align="center">
  <a href="https://make-it-count.ro">
    <img src="https://geseidl.ro/assets/icons/makeitcount-amprenta-gold.png" alt="makeitcount" height="60">
  </a>
  <br>
  <sub><em>We believe great tools should be shared. Every contribution counts.</em></sub>
  <br>
  <sub><a href="https://make-it-count.ro">make-it-count.ro</a></sub>
</p>
