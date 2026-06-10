---
description: Delegate a task to xAI Grok (default grok-4.3) for an independent fourth-family second opinion or alternative reasoning. Override with -m grok-4.20-0309-reasoning (step-by-step) or grok-build-0.1 (code).
argument-hint: <task description, optionally with file paths or -m model>
---

Forward $ARGUMENTS to xAI Grok via the `grok-rescue` subagent.

Use for:

- **Fourth-family second opinion**: adversarial review where a model family independent of Claude/OpenAI/Google reduces correlated errors. Pair with `/gemini:review` and `/codex:review` for a diverse panel.
- **Contra-opinie**: "Claude concluded X — does Grok agree, and why or why not?"
- **Alternative reasoning**: when Claude is stuck, a different reasoning style may break the impasse.

## Model selection

- Default: `grok-4.3` (flagship, alias `grok-latest`).
- `-m grok-4.20-0309-reasoning` — explicit step-by-step reasoning.
- `-m grok-build-0.1` — code-focused tasks.

## Guard the premise (mandatory for adversarial review)

When this is a second-opinion / contra-opinie pass, instruct Grok to **verify the underlying premise independently from the raw data BEFORE evaluating Claude's conclusion** — do not feed Claude's conclusion as fact. Agreement counts as confirmation only if Grok re-derived the premise itself. (Three reviewers fed the same wrong premise fail identically.)

## Currency of facts

For time-sensitive questions (versions, prices, regulations, breaking news), tell Grok to use **Live Search** and return citations rather than relying on training knowledge. See `/grok:live` for a search-first variant.

If $ARGUMENTS is empty, ask the user what they want Grok to do.
