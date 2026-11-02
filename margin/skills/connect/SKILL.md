---
name: connect
description: Connect this project's AI calls to Margin so it can meter cost per job and outcome. Use when the user says "connect Margin", "set up Margin", "meter my agents", or asks why Margin shows no data. Picks the right door (OpenRouter Broadcast, OpenTelemetry via `margin connect`, the Python/TypeScript SDK, or a GitHub repo connect link), writes no secret anywhere, and says what it did and what it does not capture.
---

# Connect to Margin

Margin needs two things from a project: its AI calls (the cost side) and its eval results
(the outcome side). This skill wires the first and points at the second. The Margin MCP
server in this plugin is already authenticated with the user's ingest key; never ask the user
to paste that key into the chat, and never write it into a file in the repo.

## 1. See what Margin already has

Call `margin_list_projects`. One row comes back: the project the key belongs to.
- `real_calls > 0`: calls are already arriving. Say so, skip to step 4.
- `real_calls == 0`: nothing has arrived yet. Pick a door in step 2.

If the tool errors with "unauthorized", the key in the plugin settings is wrong or revoked.
Tell the user to set it again with `/plugin` (Margin → configure). Stop there.

## 2. Pick the door from what the repo shows you

Look at the code before choosing. Pick the first that fits, and tell the user why:

| the repo shows | door | what you do |
|---|---|---|
| calls go through OpenRouter (`openrouter.ai` base URL) | **OpenRouter Broadcast** | print the settings below; no code change |
| the user wants Claude Code / Codex usage metered, or the app already exports OpenTelemetry | **OTLP** | have the user run `margin connect` (below) |
| direct OpenAI / Anthropic / Gemini SDK calls in Python or TypeScript | **SDK** | call `margin_instrument_plan`, make its edits |
| the user wants Margin to run face-offs on the code itself | **GitHub** | give the link below |

### OpenRouter Broadcast (no code change)

Tell the user to open OpenRouter → Settings → Observability, turn on Broadcast, add an
OpenTelemetry Collector or Webhook destination, and enter:

```
Endpoint:  https://trymargin.io/api/ingest/otlp/v1/traces
Headers:   {"Authorization": "Bearer <your Margin ingest key>"}
```

They type the key into OpenRouter themselves. You do not see it and do not write it. Tell
them to turn on Cost allocation (under Additional generation metadata) so each call carries
what OpenRouter billed, and to add `workflow_id` to each request's `trace` object so the calls
are grouped by job. Margin reads tokens, cost and labels, not prompts.

### OTLP from a coding assistant (`margin connect`)

Have the user run this in their own terminal, with the key coming from their environment,
not from the chat:

```
pip install margin-meter
export MARGIN_INGEST_KEY=…   # from their secret store
margin connect --dry-run     # shows exactly what it would write
margin connect
margin connect --check       # "first call received" once a request has gone out
```

`margin connect` keeps the key in `~/.margin/ingest_key` (mode 600) and points Claude Code at
a helper script, so `settings.json` holds a path, never the key. It does not configure
Cursor: Cursor's OTLP export lives in its team admin dashboard, and the command says so.

### SDK (Python or TypeScript)

1. Call `margin_instrument_plan` with the language and a `workflow_id` named for the job.
2. Make the edits it returns. The key is read from `MARGIN_INGEST_KEY` at runtime; put it in
   a gitignored `.env` or the user's secret store (check `.gitignore` first), never in source.
3. Run the verify command it returns (`python -m margin_meter.doctor --json`, or
   `npx margin-meter doctor --json`) and report what it printed. `doctor` lists every
   provider call site and whether the meter covers it. A site it cannot prove covered is
   reported as a gap: say each gap out loud.

### GitHub repo connect

Margin can read the repository, find the agents and their evals, and run face-offs against
them. That connection is made by the user, signed in, at
https://console.trymargin.io/me ("Connect GitHub"). Give the link. You cannot make this
connection for them, and you should not try.

## 3. Evals (the outcome side)

Without outcomes there is no cost per outcome, only spend. If the repo has an eval harness
(promptfoo, braintrust, langsmith, ragas, deepeval, pytest, langfuse), call
`margin_eval_setup_plan` and make its edits. If it has none, say so plainly; do not invent a
pass/fail signal.

## 4. Report

End with a short list: the door used, every file you changed, the command you ran to verify
and what it printed, and what is still not captured. Always name what is outside the meter:
tool and API costs that are not model calls, human time, and subscription seats. If you found
nothing to connect, say "found nothing", never "all set".
