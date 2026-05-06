---
description: Enrich a list of prospects with verified email + phone, optionally push results to Salesforce or HubSpot. Use when the user asks to "enrich", "get emails for", "verify contacts", "find phone numbers for", or "push to CRM" prospects they've already found. Chains enrich_contacts and (optionally) crm_query for verification.
---

# Enrich prospects + sync to CRM

The user has a list of prospects already (from a prior `find_prospects` / `find_person` / `find_prospects_by_career_history` call) and wants to add verified contact info — work email, optionally personal email, optionally phone.

The user's input ($ARGUMENTS) may be:
- A reference to prospects from earlier in the conversation ("enrich the top 5", "get emails for everyone you found at Acme")
- An explicit list of `leadGenProspectId`s
- A list of names + companies (in which case run `find_person` first to materialize them)

## Step 1 — Resolve the target list

You need `leadGenProspectId`s. Three paths:

a. **Already in conversation context** — if the user references "the prospects you just found", grab their IDs from the prior tool result.

b. **User gave specific IDs** — use them directly.

c. **User gave names + companies** — for each, call `find_person({ name: "...", company: "..." })` first. The tool saves a LeadGenProspect row and returns its ID. Use those.

## Step 2 — Confirm what to enrich

Before spending credits, confirm with the user:
- Work emails (default: yes — the only thing they almost always want)
- Personal emails (default: no — separately billed)
- Phone numbers (default: no — separately billed)

If the user said "everything" or "full enrichment", set all three to `true`.

## Step 3 — Run the enrichment

Call **`enrich_contacts`** with:
```
leadGenProspectIds: [<list>],
getWorkEmails: true,
getPersonalEmails: <if confirmed>,
getPhoneNumbers: <if confirmed>
```

The tool refuses already-enriched rows automatically — no double charge.

If the response is `enrichment_pending: true` with a `jobId`, the batch is large (>5 prospects) and went async. Tell the user:
> "Running enrichment in the background — jobId `<id>`. Should be done in 2–3 minutes. The results will land in your dashboard's Lead Gen AI page automatically; I can also poll for you if you want me to wait."

If sync, present the enriched results inline:

| Name | Title | Email | Phone | Status |
|------|-------|-------|-------|--------|
| ... | ... | ... | ... | ✅ enriched / ⚠️ no email found |

## Step 4 — Optional: verify against the CRM

If the user wants to know who's already in their CRM (to avoid duplicate outreach), call **`crm_query`** for each enriched prospect's email:
```
objectType: "Contact" (Salesforce) or "contacts" (HubSpot),
fields: ["Id", "Email", "AccountId", "OwnerId", "Status"],
filters: [{ field: "Email", operator: "IN", value: [<emails>] }]
```

Show:
- 📦 **In CRM already** — list, with current owner
- 🆕 **Net-new** — list

## Step 5 — Optional: push net-new contacts to CRM

If the user wants to push the net-new contacts to their CRM:
- Today the MCP **does not yet expose a write path to Salesforce/HubSpot**. Tell the user:

> "Pushing to CRM from MCP isn't wired yet — you can either (a) push from the dashboard's Lead Gen AI page where these prospects are now saved, or (b) export and bulk-import. Want me to flag this as a tool we should build next?"

## Notes for the model

- Step 1.c (find_person) charges per call. Only batch-resolve if the user clearly wants it — otherwise ask them to use prospect-account or find-and-engage flows that already produce IDs.
- Don't enrich speculatively. If the user said "find me prospects at Acme" but didn't ask to enrich, don't run this skill — just present the unenriched list and ask.
- `enrich_contacts` with `getPhoneNumbers: true` is the most expensive line item. Only flip it on with explicit user consent.
