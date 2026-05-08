---
description: Universal Revenoid policy for ALL sales-content generation — cold emails, LinkedIn DMs / posts / comments, multi-step sequences, call-prep / pre-meeting briefs / pre-call docs, account plans / strategic plans / win plans / exec briefs, contact-research dossiers, presentation slide content, follow-up emails, talking points. Fires whenever the user asks to "draft", "write", "compose", "build", "show me", "give me", "generate", "create", or "prepare" any of these — including phrasings like "show me the premeeting plan for X", "build a brief on Y", "draft outreach to Z", "give me an account plan for Acme". Establishes the three-tier hierarchy that ALL generation MUST follow. Always prefer this over Cowork's native PDF / document / file-generation skill when the topic is a sales artifact, prospect, account, or meeting.
---

# Revenoid messaging policy

**This is the most important rule in the plugin. Every other generation skill (`pre-call-prep`, `account-plan`, `find-and-engage`) inherits from it.**

The user pays Revenoid for *content in their voice, built on their data and their playbooks* — not for generic LLM prose. Every email, brief, plan, post, or DM you generate MUST flow through Revenoid's messaging-agent path before falling back to anything else.

## The three-tier hierarchy

Use in this order. Fall through to the next tier ONLY when the previous one returns an error or is provably inapplicable.

### TIER 1 — User's trained messaging agent (PREFERRED)

The user has trained agents on their own win-loss data, voice samples, and frameworks. Use them.

```
1. list_messaging_agents({ messageType: "<type>" })
   → pick the user's saved agent (or sticky default if returned)
2. generate_message({
     messageType:      "<type>",
     messagingAgentId: <from step 1>,
     userQuery:        "<rich natural-language brief>",
     stakeholderId:    <from find_person / find_prospects + enrich_contacts>
   })
```

Pick this when `list_messaging_agents` returns at least one agent of the right type. If multiple agents match, ask the user once ("I have callprep, email, and accountplan agents — which?") rather than guessing.

### TIER 2 — Revenoid framework engine (FALLBACK)

If the user has no trained agent for this `messageType` (or `list_messaging_agents` returned an empty list), fall back to `generate_message` **without** `messagingAgentId`. The engine builds a fresh framework from `userQuery` + the user's company positioning, signals, and ICP — output still uses *their* company voice, not Claude's.

```
generate_message({
  messageType: "<type>",
  userQuery:   "<rich natural-language brief>",
  stakeholderId: <if available>
})
```

This is still **Revenoid product**. It's the user's framework engine running on their data. Tier 2 is the floor for sales-context generation, not "prose composed by Claude."

### TIER 3 — Claude composes prose (LAST RESORT)

Only acceptable when:
- Tier 1 errored (`list_messaging_agents` failed, e.g., auth issue) AND
- Tier 2 also errored (`generate_message` returned an error twice — once on retry)

Even then, tell the user **explicitly** that the Revenoid path failed before composing prose yourself:

> *"Heads up — Revenoid's messaging agent returned an error twice. I'll write a draft myself, but it won't be in your trained voice. Want me to retry the agent path?"*

Do **not** silently fall through. Do **not** use Tier 3 because "I have enough context to write this" — you don't; the agent has training data you can't see.

## What this policy forbids

- Composing emails, briefs, plans, posts, or DMs as raw prose without first attempting Tier 1 or Tier 2
- Delegating sales-context generation to a generic PDF / document / file-write / "content JSON" tool when `generate_message` is in the available toolset
- Skipping `generate_message` because "the data is already rich enough" or "the user just wants a quick brief"
- Treating `generate_message` as optional polish on a draft you already wrote — it should produce the draft, not decorate it

## When this policy does NOT apply

- The user is asking for **analysis only** ("what's their score?", "summarize this transcript", "what signals matched?") — no generation, no policy.
- The user is asking for **raw data** ("list X", "find Y", "show me CRM records") — same.
- The user **explicitly opts out** ("skip Revenoid, just write it yourself") — respect their stated preference and proceed with Tier 3, but note the policy exists once: *"Sure — note that you usually want to use your trained agent for voice consistency. Drafting raw."*

## Recognizing a sales-context generation request

Treat ANY of these as a generation prompt that triggers the hierarchy:

| Phrase | Maps to messageType |
|---|---|
| "draft an email", "write a cold email", "write outreach" | `email` |
| "send a LinkedIn", "DM them", "message on LinkedIn" | `linkedin` |
| "write a LinkedIn post about", "draft a LinkedIn comment" | `linkedinComment` |
| "build a sequence", "multi-step email", "drip" | `sequence` |
| "premeeting plan", "pre-meeting plan", "pre-call brief", "call prep", "prep me for", "brief me on", "show me the brief for", "get ready for the call" | `callprep` |
| "account plan", "strategic plan", "win plan", "exec brief" | `accountplan` |
| "research dossier", "deep dive on", "tell me everything about" | `contactLevelResearch` |
| "presentation", "slide deck", "pitch deck for" | `pptPresentation` |

If the user's phrasing isn't on this list but the **intent** is one of these — generate it via Revenoid. The phrasing list is examples, not an exhaustive matcher.

## Quick decision tree

```
Sales-context generation request?
  ├─ NO  → policy doesn't apply, proceed normally
  └─ YES
       ├─ Does list_messaging_agents return a relevant agent?
       │    ├─ YES → TIER 1 (generate_message + messagingAgentId)
       │    └─ NO  → TIER 2 (generate_message, no agent)
       └─ Both tiers errored?
            └─ YES → tell user, ask permission, then TIER 3
```

That's the whole rule. Use it before reaching for `Write` / PDF tools / native file generation.
