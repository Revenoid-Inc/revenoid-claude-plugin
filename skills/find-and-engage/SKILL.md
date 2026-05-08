---
description: Discover net-new target accounts based on the user's saved ICP, then optionally find prospects + draft outreach. Use when the user says "find me leads", "show me new accounts", "find me X companies", "build a target list", or wants a fresh batch of prospects without naming a specific company. Chains get_company_info → find_accounts (or find_accounts_by_signals) → optionally find_prospects + generate_message.
---

# Find new accounts and (optionally) engage

The user wants fresh accounts to go after — they haven't named a specific company. Use their saved ICP to discover qualified targets and walk them through whether to drill deeper.

The user's input ($ARGUMENTS) may include:
- Quantity ("find me 10 SaaS companies")
- Industry / vertical override ("logistics companies in California")
- Signal hint ("companies hiring SDR managers")
- Nothing at all (just "find me leads")

## Step 1 — Confirm the ICP context

Call **`get_company_info`** ONCE at the start of the conversation (free, no credit charge). Show the user a one-line summary:

> *"Pulling from your '[ICP setting name]' ICP — [industries] · [size range] · [locations]. Tracked signals: [top 3 buying signals]."*

This grounds the search in their config. If the user gave any overrides in $ARGUMENTS, mention you'll layer them on top.

Optionally: if the user has multiple ICPs, mention `list_icp_settings` lets them switch.

## Step 2 — Choose discovery flow

Two paths. Pick based on the user's intent:

**A. Signal-driven discovery** — `find_accounts_by_signals` — when the user wants accounts matching their tracked buying / hiring / fact signals. Use this when the user mentioned signals or said things like "companies showing buying intent." Note this is **async**: returns a `rawListId` immediately and the job (2–5 min) lands results in the dashboard's Lead Gen AI page. Tell the user where to find them.

**B. Filter-driven discovery** (default) — `find_accounts` — pure ICP-filter search via Fiber. Sync, returns rows immediately. This is the right call for general "find me leads" requests.

For path B, pass:
- `numberOfAccounts: <user's count, default 10>`
- Any overrides from $ARGUMENTS (`industryOverride`, `keywordsOverride`, `nameLike`, `naicsCodes`)
- `freshness: true` (default) — excludes already-seen accounts so repeat calls return NEW results

## Step 3 — Present the accounts

For path B (sync), show a clean table:

| # | Company | Industry | Location | Headcount |
|---|---------|----------|----------|-----------|
| ... | ... | ... | ... | ... |

End with a **"What's next?"** section containing EXACTLY 3 backtick-quoted prompts on their own lines. Backtick-quoted prompts get rendered as tab-acceptable suggestions in Claude Code — each MUST be a complete, ready-to-send prompt. Reference specific accounts from the result table (by row number or name) so the suggestions are tightly contextual. Example shape:

## What's next?

- `Prospect #3 (<that row's company name>) — top 5 contacts with verified emails`
- `Research #1 — full account intel + buying signals`
- `Show me 10 more accounts with the same filters`

## Step 4 — If user picks specific accounts to engage

When the user picks one or more accounts (e.g. "prospect #2 and #5"):

For each picked account:
1. Run `find_prospects({ companyDomains: [<domain>] })` — bulk people search at that company
2. Run `enrich_contacts({ leadGenProspectIds: [...] })` — verified emails

Then if the user wants outreach drafted, follow the **`messaging-policy`** hierarchy — never compose the email as prose yourself, never delegate to a generic document tool:

3. Call `list_messaging_agents({ messageType: "email" })`.
   - If it returns at least one agent: capture `defaultAgentId` → **TIER 1**.
   - If empty: skip the agent ID → **TIER 2** (framework engine still uses the user's company voice).
4. For each enriched prospect, call `generate_message` with:
   - `messageType: "email"`
   - `messagingAgentId: <defaultAgentId>`  *(omit this field for TIER 2)*
   - `stakeholderId: <prospect's stakeholder id>`
   - `userQuery: "Cold email from <user name> at <user company> to <prospect name> at <account>. Angle: <best matched signal from research_account>."`

**Do not write the cold-email body as prose yourself.** TIER 3 (Claude prose) is only acceptable if `generate_message` errors twice — and even then, disclose the fallback to the user before composing.

Show each draft inline. Offer: "Want me to push these to your Outreach / Salesloft sequence?" (Note: that's a future capability, not yet wired.)

## Notes for the model

- Step 1 (`get_company_info`) is free — call it on EVERY find-and-engage conversation start so the user sees their ICP context.
- Default to path B unless the user explicitly mentions signals.
- `find_accounts` defaults to `freshness: true` which excludes already-saved companies. The first run on a fresh org may surface 0 results if their saved list is huge — if so, suggest `freshness: false` to see the original page-1 set.
- Don't enrich speculatively — only when the user picks specific accounts. Enrichment is the most expensive step.

## Error handling

| Error contains… | Tell the user |
|---|---|
| `INSUFFICIENT_CREDITS` / 402 | "You've used your free credits. Upgrade at https://admin.revenoid.com/p2p/billing or run `/revenoid:setup`." |
| `INVALID_API_KEY` / 401 / `revoked` | "Your API key isn't working. Mint a new one at https://admin.revenoid.com/p2p/settings. Run `/revenoid:setup` for steps." |
| `API key required` | "Run `/revenoid:setup` — the plugin can't see your API key env var." |
| `No ICP/Persona setting found` | "You don't have an ICP configured yet. Set one up at https://admin.revenoid.com/p2p/onboarding (or click 'Create ICP' from your dashboard's Lead Gen AI page)." |

Always end with a tab-acceptable recovery prompt like `/revenoid:setup` or the relevant dashboard link.
