---
name: check
description: Find which of this project's steps are worth a Margin face-off (a measured test of a cheaper or better setup against the project's own evals) and how to start one. Use when the user asks "can this be cheaper", "should I switch models", "what should I test", or "is there a cheaper model that holds up". Reads only; a face-off is started by the user in the Margin console.
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

## 4. Hand over

Queueing a face-off spends the project's measurement allowance, so it is the user's click,
in the console at the `queue_in` link `margin_optimize` returns (https://console.trymargin.io/me).
You cannot queue one from here, and you should not try another route. Give the user the link
and the exact move to pick, by its `level`, `target` and `lever`.

Never state a saving that is not `measured`. "A cheaper model might work" is a prior;
"switching `summarize` to X held 98% of passes at 40% of the cost on 120 cases" is a result.
