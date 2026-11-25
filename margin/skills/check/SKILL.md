---
name: check
description: Find which of this project's steps are worth a Margin face-off (a measured test of a cheaper or better setup against the project's own evals) and how to start one. Use when the user asks "can this be cheaper", "should I switch models", "what should I test", or "is there a cheaper model that holds up". Reads with the ingest key; with an agent key a person created, it can also ask Margin to test a move within that key's spend limit.
---

# What is worth a face-off

A face-off runs the project's own cases twice, once as it runs today and once with one change
(a cheaper model on one step, less reasoning effort, a shorter context), and compares pass
rates. Margin promotes a change only when quality holds at its bar and the cost goes down.

## 1. Read the moves

Call `margin_optimize`. Each item is a move with a `basis`:
- `measured`: a face-off already ran. `numbers` carries the result. Lead with these.
- `detected`: Margin saw the pattern in this project's own traffic.
- `prior`: other projects' evidence suggests it; this project has not tested it.
- `untested`: a named idea with no evidence. Never present it as a saving.

`actable: "pr"` is a setting a face-off can test. `actable: "structural"` (split or merge an
agent) needs a person to build first; say that and move on.

## 2. Weigh it against spend

Call `margin_jobs`. A move on a job that is 60% of spend matters more than one on a job at
1%. Rank what you recommend by the job's share of spend, and say which job each move touches.

## 3. Check it can be measured

A face-off needs a finished baseline run and evals. Call `margin_faceoff_status`:
- `run: null` means no measured run yet. The user starts one from the console first.
- If `margin_jobs` shows `evals: false` for the job, a face-off there has nothing to grade.
  Say so, and suggest `/margin:connect` step 3 (evals) before any face-off.

## 4. Test it, or hand over

Queueing a face-off spends the project's measurement allowance, so it needs a credential a person
granted for exactly that.

**If a `margin-agent` MCP server is connected** (an agent key, `mak_…`, that a person created in
the console under Settings > Agent keys), call `margin_test_move` with the move's `level`,
`target` and `lever`. It queues the face-off within the key's spend limit, or returns a named
refusal. Then read the verdict with `margin_faceoff_status` once the run finishes. The verdict is
Margin's gate, not yours: report it as given, refusals included, and do not re-judge the numbers.
A passing change still becomes a pull request only when a person opens it in the console.

**Otherwise**, the ingest key this plugin uses can read but never spend. Give the user the
`queue_in` link `margin_optimize` returns (https://console.trymargin.io/me) and the exact move to
pick, by its `level`, `target` and `lever`. Do not look for another route. To let you request
tests yourself, the user creates an agent key with a spend limit in the console and connects it:

    claude mcp add --transport http margin-agent https://console.trymargin.io/api/mcp/agent \
      --header 'Authorization: Bearer ${MARGIN_AGENT_KEY}'
