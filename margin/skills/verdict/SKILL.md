---
name: verdict
description: Get Margin's verdict on one specific change before you make it, a named model, an eval export to replay, or a candidate a new release just shipped, and open the pull request only if Margin promotes it. Use when you or the user are about to "try this model", "replay my evals", ask whether it "holds up", "is this eval set big enough", or "test the new release on our cases". Needs an agent key for anything that spends.
---

# Ask Margin before you change a step

You propose a change; Margin decides whether it holds on the project's own cases, with a bar
that corrects for how many candidates were tried. Do not grade the change yourself: an agent
that tries several models and keeps the best-looking one has found noise as often as a win.

## 1. Is there enough to decide on?

If you have an eval export (promptfoo results, a Braintrust or LangSmith export, or JSONL with
an input and an expected answer), call `margin_fit_check` with it first. It is free and answers
in seconds: how many graded cases the step has, and how many the gate needs for the number of
candidates you plan to try. If it says not ready, tell the user how many more cases are needed
and stop. Do not run a replay that cannot promote.

## 2. Get the verdict

Pick the path that fits what the step is:

- **A single model call with an eval export** (classify, extract, route, answer): call
  `margin_replay_dataset` with the dataset, the `baseline` model it uses today, one `candidate`
  and a `cap_usd`. Margin runs both on the same inputs, grades them with the export's own checks
  and returns the verdict. No repo is run.
- **An agent or repo Margin already measures** (a finished baseline run shows in
  `margin_faceoff_status`): call `margin_test_model` with the candidate `model`. Margin queues
  a face-off within the key's spend limit. Read the result later with `margin_faceoff_status`.

Try one candidate at a time and say up front which ones you mean to try. Every candidate tried
on the same cases raises the bar for all of them, and Margin counts them.

## 3. Report it as given

The verdict is `promoted`, `held` or a named refusal. Report it with its numbers: cases each side
passed, cases lost, the cost change, and the yearly figure if it holds at the volume given. For
`held`, say what it would take (`cases_to_settle`, or the cases it lost). Never round a hold up
to a win, and never re-run the same comparison hoping for a different answer.

## 4. Act only on a promotion

Open the change as a pull request only when the verdict is `promoted`, and say in its description
which Margin verdict it rests on. A person merges it. After the merge, Margin re-measures on the
next run; check `after_merge` in `margin_faceoff_status` and report if the change stopped holding.

If no `margin-agent` server is connected, the ingest key can run `margin_fit_check` only. To let
you request verdicts, the user creates an agent key with a spend limit (Settings > Agent keys) and
connects it:

    claude mcp add --transport http margin-agent 'https://console.trymargin.io/api/mcp/agent?tools=core' \
      --header 'Authorization: Bearer ${MARGIN_AGENT_KEY}'
