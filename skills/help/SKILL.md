---
description: Show example prompts and tips for using the Revenoid plugin. Manual-only — user must explicitly type /revenoid:help.
disable-model-invocation: true
---

# Revenoid plugin — quick reference

When the user invokes `/revenoid:help`, present the content below VERBATIM as a markdown reply. Don't paraphrase, don't expand, don't add your own commentary — just print this catalogue. The user is asking *what can I do?* and wants a scannable list of working prompts they can copy.

If the user passed any text after `/revenoid:help` (in `$ARGUMENTS`), and it matches one of the section names below (e.g. "discovery", "calendar", "crm", "outreach"), filter to just that section. Otherwise show everything.

---

## 🚀 Revenoid plugin — what you can do

Five **task skills** auto-fire based on what you say. You can also call any of the **23 tools** directly. Free plan ships with 500 credits.

### 🎯 Discover & prospect

Try these:

```
Prospect Ascendion for me — top 5 contacts with verified emails
Find me 10 cybersecurity SaaS companies in the US
Who at Snowflake is in RevOps but came from an IC sales background?
Is Gong hiring SDR managers right now?
```

Skills involved: `prospect-account`, `find-and-engage`. Tools used behind the scenes: `lookup_company`, `research_account`, `find_prospects`, `find_prospects_by_career_history`, `find_job_postings`, `enrich_contacts`.

### 📅 Pre-meeting prep

```
Brief me on my next call
Pre-call prep for tomorrow's demo with Acme
Who at acme.com have I met with in the last 60 days?
```

Skills involved: `pre-call-prep`. Tools: `get_calendar_events`, `get_calendar_stakeholders`, `find_person`, `lookup_linkedin_posts`, `search_call_transcripts`, `generate_message(callprep)`.

### 📋 Account planning

```
Build a strategic account plan for OpenAI
Generate an account plan for Stripe using my "Enterprise SaaS" agent
```

Skill involved: `account-plan` (async — takes 3–7 minutes; the plugin polls automatically via `get_job_status`).

### 💼 CRM + calendar queries

```
Pull my last 10 closed-lost opportunities from Salesforce with the close reason
Show contacts at HubSpot accounts where deal value > $50k
What meetings do I have today?
```

Tools: `crm_query`, `get_calendar_events`.

### ✍️ Outreach drafting

```
Draft a cold email to Stephanie Eldred at Ascendion using my SDR email agent
Generate a LinkedIn DM for the CTO of Anthropic
Write a follow-up sequence for accounts that ghosted after the demo
```

Tools: `list_messaging_agents` (find your trained agents), `generate_message`.

### 🔍 Quick lookups

```
Look up ascendion.com — who are they?
Look up stephanie@ascendion.com — what's her role?
Find Stanford alumni currently at Anthropic in the Bay Area
```

Tools: `lookup_company`, `lookup_email`, `lookup_person` (the most flexible — accepts any combination of name, email, LinkedIn slug, school, location, job title).

### 🛠 Power-user — direct tool calls

```
Use the revenoid find_accounts tool with industryOverride="Logistics" and numberOfAccounts=20
Call lookup_person with linkedinSlug="satya-nadella" and getDetailedWorkExperience=true
Run research_account on "Toyota Financial Services"
```

You can call any of the 23 tools by name; the plugin's MCP server handles the rest.

---

## 💡 Tips

- **Switch ICPs** — say *"switch to my SDR Manager ICP"* — runs `list_icp_settings` then scopes the next `find_accounts` call to that setting.
- **Pick a specific messaging agent** — say *"using my callprep agent"* or *"using my Q4 demo prep agent"* — runs `list_messaging_agents` and passes the chosen `messagingAgentId` to `generate_message`.
- **Async generations** — account plans + PPT presentations take 3–7 min. The plugin returns a `jobId` immediately; Claude polls `get_job_status` automatically.
- **Free first** — `get_company_info`, `search_call_transcripts`, `crm_query`, `get_calendar_events`, all `list_*` tools are free. Fiber/Coresignal/Apollo lookups + ML generations are paid.
- **Save your key permanently** — add to `~/.zshrc`:
  ```
  export REVENOID_API_KEY="rvk_live_..."
  ```

---

## 📚 More

- 23 full tool list: `/revenoid:help tools` (or visit https://revenoid.com/mcp)
- Plugin source: https://github.com/Revenoid-Inc/revenoid-claude-plugin
- Get a key / buy credits: https://admin.revenoid.com → Settings → API Keys

---

## What's next?

End the help output with EXACTLY 3 backtick-quoted prompts on their own lines. Backtick-quoted prompts get rendered as tab-acceptable suggestions in Claude Code, so each MUST be a complete, ready-to-send prompt the user could fire as their next turn. Pick three good "first run" prompts that show off the breadth of the plugin without being too specific:

## What's next?

- `Show me my saved messaging agents and ICP settings`
- `Find me 10 accounts that match my ICP`
- `Brief me on my next call`

---

## ⚙️ Other slash commands

If the user typed `/revenoid:help tools`, instead of the catalogue above, list all 23 tools grouped by category exactly as in the README:

**Discover & qualify** (7): get_company_info · find_accounts · find_accounts_by_signals · find_prospects · find_prospects_by_career_history · find_person · research_account
**Lookups** (4): lookup_person · lookup_company · lookup_email · lookup_domain
**Enrichment & signals** (4): enrich_contacts · lookup_linkedin_posts · find_job_postings · search_call_transcripts
**Calendar** (2): get_calendar_events · get_calendar_stakeholders
**CRM** (1): crm_query
**Outreach generation** (2): list_messaging_agents · generate_message
**Resources** (2): list_saved_sequences · list_icp_settings
**Async polling** (1): get_job_status

If the user typed `/revenoid:help commands` (or asked about all commands), list the full slash-command surface:

| Command | What it does |
|---|---|
| `/revenoid:help` | This help screen |
| `/revenoid:help tools` | Flat tool catalogue |
| `/revenoid:help commands` | This command list |
| `/revenoid:credits` | Show credit balance + plan |
| `/revenoid:agents [type]` | List your saved messaging agents (optional type filter: email, callprep, accountplan, etc.) |
| `/revenoid:icp [name]` | Show ICP settings; pass a name to switch which ICP scopes the next call |
| `/revenoid:prospect-account <company>` | Full prospect workflow: research + enriched contact list |
| `/revenoid:find-and-engage [filters]` | Discover net-new accounts via your ICP |
| `/revenoid:pre-call-prep [meeting]` | Calendar-aware briefing for your next call |
| `/revenoid:account-plan <company>` | Async strategic account plan generation |
| `/revenoid:enrich-and-sync` | Bulk enrich + CRM duplicate check |
