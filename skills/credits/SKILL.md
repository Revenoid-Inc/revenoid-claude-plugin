---
description: Show the user's current Revenoid credit balance and plan. Manual-only — user must explicitly type /revenoid:credits.
disable-model-invocation: true
---

# Credits & plan

User invoked `/revenoid:credits`. Pull current account state and present it cleanly.

## Step 1 — Fetch the data

The full credit / plan picture is on the user's account record but isn't surfaced by `get_company_info` directly. The plan + plan_status + credit balance are visible to the user in the dashboard's `/p2p/billing` page. Since there's no MCP tool that reads credits today, **be honest with the user**:

> *Credit balance + plan details aren't yet exposed via MCP — I can't read them from here. To see them right now:*
>
> *- Open https://app.revenoid.com/p2p/billing to see your remaining balance, plan tier, and usage breakdown by category*
> *- Or, if you'd like a CLI-style answer, you can run a free tool to confirm your account is connected:*

## Step 2 — Confirm the account is connected

Call **`get_company_info`** (free, no credit charge). Show the user:
- Their resolved company name + ICP setting (proves the API key is valid + connected)
- Their connected integrations
- Tracked signal count

## Step 3 — Present + suggest next moves

End with a **"What's next?"** section containing EXACTLY 3 backtick-quoted prompts on their own lines. Backtick-quoted prompts render as tab-acceptable suggestions in Claude Code:

## What's next?

- `Show me my saved messaging agents`
- `Switch my active ICP setting`
- `Find me 10 net-new accounts using my current ICP`

## Notes for the model

- Don't fabricate credit numbers — if the data isn't available via MCP, say so.
- This skill is intentionally short. Most users wanting "how am I doing on credits" want a one-line answer + the dashboard link, not a deep dive.
