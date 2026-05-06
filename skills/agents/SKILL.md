---
description: List the user's saved Revenoid messaging agents (cold email, callprep, account plan, LinkedIn comment, etc). Manual-only — user must explicitly type /revenoid:agents.
disable-model-invocation: true
---

# Saved messaging agents

User invoked `/revenoid:agents`. Show their trained Messaging Agents grouped by type so they can pick which one to use in subsequent `generate_message` calls.

## Step 1 — Fetch the agents

Call **`list_messaging_agents`** with `preferDefault: false` so the user sees the FULL list (no sticky-default reordering). If the user passed an argument (`$ARGUMENTS`), use it as the `messageType` filter — accepts `email`, `linkedin`, `sequence`, `accountplan`, `callprep`, `contactLevelResearch`, `pptPresentation`, `linkedinComment`, OR a free-text substring search via `q`.

## Step 2 — Present grouped by type

Format as a markdown table grouped by `messageType`:

```
### 📧 Email agents (N)
| Name | ID | Created |
|------|----|----|
| ...  | ... | ... |

### 📞 Call-prep agents (N)
...
```

For each agent, show:
- Friendly name (`name` field)
- `messagingAgentId` (truncate to first 8 chars for readability + tail with `…`)
- Created date (relative — "3d ago" / "2mo ago")

If `defaultAgentId` is set in the response, mark that agent with ⭐ in its row.

If the user has only system / seed agents (`isSystemAgent: true` for all), call that out:

> *You don't have any custom-trained agents yet. The agents below are Revenoid-provided defaults. Train your own at https://admin.revenoid.com/messaging-ai/messaging-agent for tighter voice match.*

## Step 3 — Suggest next moves

End with a **"What's next?"** section containing EXACTLY 3 backtick-quoted prompts on their own lines. Pick contextual examples (reference an actual agent name from the list):

## What's next?

- `Draft a cold email to <prospect> using my "<agent name>" agent`
- `Build an account plan for <Company> using my "<accountplan agent name>" agent`
- `Show me only my email agents`

## Notes for the model

- Default `numProfiles`/limit is 20 — if the user has 50+ agents, mention they can pass a search term: `/revenoid:agents email` filters to email-type agents.
- Don't auto-call generate_message from this skill — this is a list-and-pick workflow, not an action workflow.
