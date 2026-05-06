---
description: Build a strategic account plan for a target company using the user's trained accountplan agent. Use when the user asks for an "account plan", "strategic plan", "exec brief", "win plan", or "play for" a specific company. Async — the generation runs 5–10 min on the ML pipeline; chains research_account → list_messaging_agents → generate_message(accountplan) and polls get_job_status until done.
---

# Strategic account plan

The user wants a deep, multi-page strategic plan for a target company. This is one of the heavyweight workflows — `generate_message(accountplan)` is async and runs 5–7 minutes through the ML pipeline.

The user's input ($ARGUMENTS) should be a company name. If unclear, ask first.

## Step 1 — Ground the plan in real signals

Call **`research_account`** with `companyName: "$ARGUMENTS"` (or the resolved name).

Capture the score, grade, matched signals, and the top 3 cited insights. This becomes the factual scaffolding for the plan.

If the score is below 40 OR no signals matched, warn the user:
> "Heads up — [Company] only scored [X]/100 with [no | weak] signals. The plan will be thin. Want to proceed anyway, or pick a higher-scoring account?"

## Step 2 — Pick the messaging agent

Call **`list_messaging_agents`** with `messageType: "accountplan"`.

If `defaultAgentId` is set, use that — the user has a sticky preference.
If no default, show the user the available `accountplan` agents and ask which to use, OR auto-pick the one whose name best matches their context.

Capture the chosen `messagingAgentId`.

## Step 3 — Kick off the generation (async)

Call **`generate_message`** with:
```
messageType: "accountplan"
messagingAgentId: <from step 2>
userQuery: "Strategic account plan for <Company>. Score <X> with signals: <list from step 1>. Sales leadership: <if known from step 1>. Tech stack: <if known>. Headcount ~<count>. Focus on strategic GTM position, top 3 plays, and 3 immediate-verification questions for the first call."
```

This will return immediately with `generation_pending: true` and a `jobId`. Tell the user:

> "🚀 Account plan generation kicked off (jobId: `<id>`). These typically take 3–7 minutes. I'll check back in 30 seconds and keep polling until it's ready."

## Step 4 — Poll for completion

Call **`get_job_status`** with the `jobId` after 30s. If `status === 'running'`, wait 30s and call again. Repeat.

If `status === 'failed'`, show the user the error and offer to retry.
If `status === 'timeout'` (job ran past 10 min), explain and suggest re-running.

When `status === 'completed'`, the full plan is in `result`.

## Step 5 — Present the completed plan

Format the plan response from `result` into a clean, scannable structure. The accountplan output typically includes:

- **Company facts** — what they do, revenue, headcount, leadership
- **Recent signals** — buying / hiring / news with cited links
- **Sales leadership** — names + titles for outreach
- **Tech stack**
- **Initial hypothesis** — pain points + Revenoid angle
- **3 recommended plays** — with cited reasoning

Mention the plan is saved to the dashboard's Generated Outputs section, then end with a **"What's next?"** section containing EXACTLY 3 backtick-quoted prompts on their own lines. Backtick-quoted prompts render as tab-acceptable suggestions in Claude Code — each MUST be a complete, ready-to-send prompt. Pick three follow-ups that build on the plan you just generated (refer to the actual leaders / signals named in the plan):

## What's next?

- `Prospect <Company> — top 5 contacts with verified emails`
- `Brief me on <Sales Leader from the plan> before reaching out`
- `Pull the last 30 days of LinkedIn posts from <CEO from the plan>`

## Notes for the model

- Step 4's polling: don't busy-loop. Call `get_job_status` once every 30 seconds — Claude Code can run other tools or chat in between if the user wants.
- If the user has more than one `accountplan` agent, mention that and offer to use a specific one ("I have callprep, accountplan, and PPT presentation agents — which would you like for this?").
- Do NOT call `generate_message(accountplan)` twice for the same company in the same conversation — it caches inside the in-app workspace, and the result from one call serves multiple requests. If the user asks to "regenerate", explain that's a different request and confirm they want a fresh run.

## Error handling

| Error contains… | Tell the user |
|---|---|
| `INSUFFICIENT_CREDITS` / 402 | "Account plan generation needs more credits than you have. Upgrade at https://app.revenoid.com/p2p/pricing — Pro plan is $100/mo for 11K credits, comfortable for 30+ account plans/mo." |
| `INVALID_API_KEY` / 401 / `revoked` | "Your API key isn't working. Run `/revenoid:setup`." |
| `API key required` | "Run `/revenoid:setup` — the plugin can't see your API key env var." |
| `enrichment_required` (rare for accountplan) | "The agent needs a researched stakeholder. This shouldn't happen for accountplan — try again, or report at https://github.com/Revenoid-Inc/revenoid-claude-plugin/issues" |
| `status: 'failed'` from get_job_status | Surface the `error` field from the job + offer "try again with a different agent" — agent issues sometimes resolve when the user picks a different `accountplan` agent. |

Always end with a tab-acceptable recovery prompt.
