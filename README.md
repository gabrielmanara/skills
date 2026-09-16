# skills

Claude Code skills, published as a plugin marketplace.

## Install

```
/plugin marketplace add gabrielmanara/skills
/plugin install mate@gabrielmanara
```

## mate

Routing doctrine for the architect-as-orchestrator pattern.

The session owns architecture, specs, routing and verification.
Everything else goes to the cheapest lane that can do the job: Haiku for exploration, Sonnet for routine implementation, Opus for judgment-heavy work, and GPT-5.6 through the Codex CLI when a second vendor's hands are worth having.
Every deliverable is then reviewed by a vendor that did not write it, plus a fresh-context advisor, before the session reports done.

Invoke it with `/mate`, or let it load when you delegate implementation work.

### Prerequisites

The Anthropic lanes need nothing beyond Claude Code.

The two cross-vendor lanes and the cross-vendor review need the [Codex CLI](https://github.com/openai/codex) installed and authenticated, on an account with access to the `gpt-5.6-luna` and `gpt-5.6-sol` models.

Without it those lanes report `STATUS: unavailable` and stop.
They never quietly fall back to a Claude model, because a cross-vendor lane that silently becomes a single-vendor lane defeats the reason you routed to it.
The rest of the doctrine still works.
