---
description: Research a target company end-to-end and surface qualified prospects with verified contacts. Use when the user wants to "prospect", "research", "find leads at", "go after", or "build a list for" a specific named company. Chains lookup_company → research_account → find_prospects → enrich_contacts and returns a ranked contact list with reasoning.
---

# Prospect a target account

When the user names a company they want to prospect, run this end-to-end workflow.

The user's input ($ARGUMENTS) will be a company name, domain, or LinkedIn URL — sometimes with extra context like "their RevOps lead" or "VP+ in marketing." Use everything they gave you.

## Step 1 — Resolve the company identity

Call **`lookup_company`** with whatever identifier the user provided:
- If they gave a domain (e.g. "acme.com") → `companyDomain: "acme.com"`
- If a LinkedIn URL → extract the slug → `companyLinkedinSlug: "acme"`
- If just a name → `companyName: "Acme"`

Capture the canonical `companyName`, `bestDomain`, `linkedin_primary_slug`, and headcount from the response. If multiple candidates come back, pick the highest-confidence match and tell the user which you chose.

## Step 2 — Score the account

Call **`research_account`** with `companyName` (or the resolved name from step 1).

Show the user:
- **Score** (0–100) and **grade** (A–F)
- **Matched signals** — buying / hiring / fact triggers Fiber found
- **Top 3 cited insights** with source links

If score is below 50, flag it: "This isn't a strong fit on current signals — want me to keep going anyway?"

## Step 3 — Find the right people

Call **`find_prospects`** with `companyDomains: [<bestDomain from step 1>]`.

If the user specified persona context ("RevOps", "VP Sales", etc.) pass it as `titlesOverride: [...]`. Otherwise the tool uses the user's saved persona settings.

Default `numberOfProspects` to 15 unless the user asked for more.

## Step 4 — Enrich the top contacts

For the most-relevant 5–10 prospects from step 3, collect their `leadGenProspectId`s and call **`enrich_contacts`** with:
```
leadGenProspectIds: [<ids>],
getWorkEmails: true
```

This adds verified work emails. Don't fetch personal emails or phones unless the user asked — those are separately billed.

If `enrich_contacts` returns `enrichment_pending: true`, tell the user the enrichment is running async and they can ask back in 1–2 minutes.

## Step 5 — Present a ranked summary

Once enriched, give the user a clean table:

| # | Name | Title | Email | Why this person |
|---|------|-------|-------|-----------------|
| 1 | ... | ... | ... | One-line fit reasoning, ideally tying back to a signal from step 2 |

End with:
- A pointer back to the `score` and the strongest signal
- A **"What's next?"** section with EXACTLY 3 backtick-quoted prompts on their own lines. Backtick-quoted prompts get rendered as tab-acceptable suggestions in Claude Code, so each one MUST be a complete, ready-to-send prompt the user could fire as their next turn. Pick the three most useful follow-ups for the prospect list you just produced. Example shape (adapt content to actual results):

  ## What's next?

  - `Draft cold emails to the top 3 using my SDR email agent`
  - `Pull LinkedIn posts for prospect #1 to find a personalized angle`
  - `Build a strategic account plan for <Company>`

## Notes for the model

- Run steps 1–4 sequentially — each depends on the previous.
- If `lookup_company` fails or returns 0 results, fall back to `research_account({ companyName: $ARGUMENTS })` and skip step 1.
- All four tools charge credits. Don't run them speculatively — only when the user has clearly named a target company.
- If the user says "prospect" without naming a company, ask them to specify.

## Error handling (applies to every step)

If any tool call returns an error, recognize the pattern and route the user to the right recovery — DON'T just surface a raw error message and stop:

| Error contains… | Diagnosis | Tell the user |
|---|---|---|
| `INSUFFICIENT_CREDITS` or HTTP 402 | Out of free credits | "You've used your free credits. Upgrade at https://admin.revenoid.com/p2p/billing — Starter plan is $20/mo for 2,000 credits. Or run `/revenoid:setup` for the full breakdown." |
| `INVALID_API_KEY` / `API key has been revoked` / 401 | Key is bad or revoked | "Your API key isn't working — likely revoked or rotated. Mint a new one at https://admin.revenoid.com/p2p/settings and update your `REVENOID_API_KEY` env var. Run `/revenoid:setup` for step-by-step." |
| `API key required` (no auth header) | Env var not set | "Run `/revenoid:setup` — looks like the plugin's MCP server can't see your API key." |
| Anything else | Unknown error | Surface the error message verbatim + suggest running `/revenoid:setup` to verify connectivity. |

After surfacing the error, ALSO offer the recovery path as a tab-acceptable suggestion:

```
- `/revenoid:setup`
- `Open https://admin.revenoid.com/p2p/billing`
```
