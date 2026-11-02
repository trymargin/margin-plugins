---
name: costs
description: Show where this project's AI money goes, per job, from Margin's meter. Use when the user asks "what are my agents costing", "which step is most expensive", "where does the money go", or wants cost per outcome. Reads only; never estimates a number the meter did not record.
---

# Where the money goes

Every number here comes from a Margin MCP tool. Never fill a gap with an estimate; if a
figure is missing, say it is missing and why.

1. Call `margin_jobs`. It returns each job (a model-calling step) over the last 30 days: calls,
   `cost_usd`, the live model, whether any eval has exercised it, and `concentration` (the
   share of spend in the top 5 / 10 / 20 jobs).
2. Call `margin_portfolio_economics` for the project totals: spend, passed outcomes and cost
   per passed outcome. `blended_cost_per_outcome: null` means no outcomes were recorded, so
   there is spend but no cost per outcome yet.
3. Call `margin_blind_spots` for what the numbers cannot see yet.

Show the user:
- a table of the top jobs by cost: job, model, calls, cost, evals yes/no;
- the concentration line, e.g. "the top 5 jobs are 82% of spend";
- total spend and cost per passed outcome, or "no outcomes recorded, so no cost per outcome";
- each blind spot with its fix.

Rules:
- `seen: "traffic only"` means the meter saw the job but no code was imported to match it.
  That is normal for a project that has not connected GitHub. Do not call it an error.
- Outcomes are recorded per workflow, not per job. Never divide a workflow's outcomes across
  its jobs to make a per-job cost per outcome.
- Value per outcome exists only where the customer supplied a value. If none was supplied,
  say the figure is cost per outcome, and that it is the honest number without a value.
- Costs outside model calls (tool APIs, human time, seats) are not in these figures unless the
  project reported them. Say so when the user asks about total cost.

If every tool returns zero calls, the project is not connected yet: suggest `/margin:connect`.
