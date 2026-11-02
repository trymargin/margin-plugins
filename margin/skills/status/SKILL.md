---
name: status
description: Read the verdicts of this project's Margin face-offs and the post-merge watch on changes already shipped. Use when the user asks "did the face-off finish", "what did Margin find", "is the model change I merged holding up", or "should I revert". Reads only; pull requests open and revert from the Margin console.
---

# Face-off verdicts and the post-merge watch

Call `margin_faceoff_status`. It returns:
- `run`: the latest measured run (the baseline every face-off is compared against).
- `faceoffs`: each face-off queued against that run, with `status`, the setup it tested
  (`model_override`, `scope`, `plan`), and `verdict` once it is done.
- `changes`: each change the user registered after merging, with its watch.

## Report the face-offs

For each face-off, one line: what it tested, its status, and the verdict in plain words.
- `queued` / `running`: say where it is (`queue_position`, `wait_note` if present).
- `done` with a verdict: quote the verdict's pass rates and cost as given. Do not round a
  "held" into "better", and do not call a result that failed the bar "close enough".
- `failed`: give `reason` as written.
- `pr_url` set: the pull request is open. Link it.

If the verdict says a change is worth making and no pull request is open, the user opens it
from https://console.trymargin.io/me. That click is their approval, so you do not open it,
and you do not reproduce the change by hand as a way around that.

## Report the post-merge watch

For each change: the step (`operation`), `before` → `after`, and the watch `status`.
- `holds`: the lower bound on accuracy since the switch is at or above the bar.
- `slipped`: the upper bound fell below the bar on enough graded outcomes. Say it plainly and
  point to the console, where Margin prepares the revert pull request (`revert_url` once open).
- `watching`: not enough evidence either way. Give `accuracy.graded` and any `note` (for
  example, calls still running on the old model). Do not call it holding.
- `not_merged`: the pull request has not merged yet, so nothing is being watched.

Give the cost line too: `cost.actual_per_call` against `cost.predicted_per_call` when both are
present.

If `faceoffs` and `changes` are both empty, say "no face-offs or watched changes yet" and
suggest `/margin:check`. Never report an empty list as "all clear".
