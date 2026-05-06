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
- An offer: "Want me to draft an email to the top 3 using your messaging agent?"

## Notes for the model

- Run steps 1–4 sequentially — each depends on the previous.
- If `lookup_company` fails or returns 0 results, fall back to `research_account({ companyName: $ARGUMENTS })` and skip step 1.
- All four tools charge credits. Don't run them speculatively — only when the user has clearly named a target company.
- If the user says "prospect" without naming a company, ask them to specify.
