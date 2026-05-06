---
description: Show the user's saved ICP / Persona settings and active configuration. Manual-only — user must explicitly type /revenoid:icp.
disable-model-invocation: true
---

# ICP & Persona settings

User invoked `/revenoid:icp`. Surface their saved ICP / Persona configurations so they can see what filters drive `find_accounts` and `find_prospects` calls — and switch between them if they have more than one.

If the user passed an argument (`$ARGUMENTS`) like `/revenoid:icp <name>`, treat it as a request to switch to that ICP for the next call (see Step 4).

## Step 1 — Get the active ICP context

Call **`get_company_info`** first (free) — its `icp_setting.name` field tells you which ICP is currently active.

## Step 2 — List all saved ICPs

Call **`list_icp_settings`** to get the full set of configurations the user has saved.

## Step 3 — Present

Show:

> ### ⭐ Active: "<active ICP name>"
>
> | Field | Value |
> |---|---|
> | Industries | <list> |
> | Locations | <list> |
> | Company size | <range> |
> | Technologies | <list> |
> | Keywords | <list> |
>
> ### Target persona
> Titles: <list>
>
> ### 📋 Other saved ICPs (N)
> | Name | Industries | Sizes |
> |------|------------|-------|
> | ... | ... | ... |

If the user only has one ICP, skip the "Other saved ICPs" section.

If any field is empty (e.g. keywords:[], description:''), call it out as an opportunity:

> *Heads up — your keywords field is empty. Adding 2–3 keywords (e.g. "AI sales", "outbound automation") tightens find_accounts results meaningfully.*

## Step 4 — Switch ICP (if argument passed)

If `$ARGUMENTS` was supplied, attempt to match it case-insensitively against an ICP name from `list_icp_settings`. If exactly one matches:

> *Got it — your next find_accounts / find_prospects call will use "<matched name>" (settingId: <id>).*

The matching is local to this conversation — Claude will pass `settingId` or `settingName` on the next discovery call. If no match (or multiple), show the available ICP names and ask the user to be more specific.

## Step 5 — Suggest next moves

End with a **"What's next?"** section containing EXACTLY 3 backtick-quoted prompts on their own lines. Customize based on what they have:

## What's next?

- `Find me 10 accounts using my current ICP`
- `Switch to my "<other ICP name>" ICP`
- `Update my ICP keywords at https://admin.revenoid.com/research-ai/signals-pitch`

If the user only has 1 ICP, replace prompt #2 with `Find prospects at <some company> with my current persona settings`.

## Notes for the model

- ICP edits aren't supported via MCP yet — you can only READ ICPs, not modify them. Direct the user to the dashboard for changes.
- Multiple ICPs is uncommon — most users have one. The "switch" path matters most when they DO have more.
