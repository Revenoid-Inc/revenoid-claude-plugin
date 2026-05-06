# Revenoid — AI Sales Workflow Plugin for Claude Code

> Discover, research, enrich, and engage prospects through 23 unified tools spanning Salesforce, HubSpot, Google Calendar, Microsoft 365, Gong, Chorus, Avoma, Coresignal, Apollo, and Fiber AI — with built-in Messaging Agents that draft outreach in your voice.

## Install

```bash
claude /plugin install revenoid
```

Once installed, set your Revenoid API key:

```bash
export REVENOID_API_KEY=rvk_live_xxxxxxxxxxxxxxxxxxxxxxxxxxxx
```

(Get a key at [app.revenoid.com](https://app.revenoid.com) → Settings → API Keys. Free plan ships with 500 credits.)

Then start Claude Code. The plugin auto-loads.

## What you can do

The plugin ships with five **skills** that Claude Code invokes automatically based on what you describe:

| Say something like… | Skill that fires | What happens |
|---|---|---|
| *"Prospect Anthropic for me"* | `prospect-account` | Resolves the company → scores it → finds prospects → enriches with verified emails → returns ranked contact list |
| *"Brief me on my next call"* | `pre-call-prep` | Pulls the meeting from your calendar → researches the company + attendees → searches past Gong/Chorus calls → generates a callprep brief |
| *"Find me 10 cybersecurity SaaS companies"* | `find-and-engage` | Discovers net-new accounts based on your saved ICP → optionally finds prospects + drafts outreach |
| *"Build an account plan for OpenAI"* | `account-plan` | Researches the account → kicks off async accountplan generation via your trained agent → polls until done |
| *"Enrich the top 5 prospects you found"* | `enrich-and-sync` | Adds verified work email (+ optionally phone, personal email) → checks against your CRM for duplicates |

You can also call the underlying 23 tools directly if you want fine-grained control.

## The 23 tools

### 🎯 Discover & qualify (7)
- `get_company_info` — your saved ICP, persona, integrations, signals (free)
- `find_accounts` — discover NEW companies via Fiber, scoped to your ICP
- `find_accounts_by_signals` — UrgencyIQ signal-based discovery (async)
- `find_prospects` — bulk people search at target companies
- `find_prospects_by_career_history` — filter on past roles (e.g. "RevOps leaders who came from sales")
- `find_person` — single named-person lookup (Apollo + Fiber)
- `research_account` — score (0–100), grade (A–F), buying-signal analysis

### 🔍 Lookups (4)
- `lookup_person` — flexible person lookup (any combination of identifiers)
- `lookup_company` — flexible company lookup (any combination of identifiers)
- `lookup_email` — email → person profile
- `lookup_domain` — domain → company identity

### 📈 Enrichment & signals (4)
- `enrich_contacts` — verified work email + optional personal email + phone
- `lookup_linkedin_posts` — recent LinkedIn posts (Coresignal)
- `find_job_postings` — open roles at a company (buying-signal probe)
- `search_call_transcripts` — search Gong / Chorus / Avoma transcripts (free)

### 📅 Calendar intelligence (2)
- `get_calendar_events` — your Google Calendar / Microsoft 365 schedule
- `get_calendar_stakeholders` — find calendar attendees from a target domain

### 💼 CRM (1)
- `crm_query` — query Salesforce or HubSpot (auto-detects which is connected)

### ✍️ Outreach generation (2)
- `list_messaging_agents` — list your trained messaging agents
- `generate_message` — draft email / LinkedIn / call-prep / account plan / sequence using YOUR voice

### 📦 Resources (2)
- `list_saved_sequences` — your outreach sequences across CRMs
- `list_icp_settings` — switch between saved ICPs

### ⏱️ Async polling (1)
- `get_job_status` — poll heavy generations (accountplan, pptPresentation)

## Pricing

All 23 tools share your Revenoid credit pool — same as the in-app workspace. The Free plan ships with 500 credits to try the full surface.

| Tool category | Cost |
|---|---|
| `get_company_info`, `search_call_transcripts`, `crm_query`, `get_calendar_events`, `list_*` | Free — Mongo / OAuth lookups |
| `find_accounts`, `find_prospects`, `find_person`, `research_account`, `lookup_company`, `lookup_domain` | Per-call — Fiber AI |
| `lookup_person`, `lookup_email` | Per-call — Fiber AI |
| `enrich_contacts` | Per channel (work email / personal email / phone) |
| `lookup_linkedin_posts`, `find_job_postings`, `find_prospects_by_career_history` | Per-call — Coresignal |
| `generate_message` (any type) | Per-call — ML inference |

Buy more credits at [app.revenoid.com/p2p/pricing](https://app.revenoid.com/p2p/pricing).

## How it works

```
Claude Code  →  npx @revenoid/mcp-server (stdio)  →  HTTPS  →  core.revenoid.com/api/v2/mcp  →  23 tools
```

The plugin runs a tiny stdio proxy locally (via `npx @revenoid/mcp-server`) that forwards every JSON-RPC tool call to Revenoid's MCP endpoint over HTTPS, attaching your API key. No data ever leaves your laptop except through that one authenticated channel.

## Configuration

Two environment variables:

| Var | Required | Default | Notes |
|---|---|---|---|
| `REVENOID_API_KEY` | yes | — | `rvk_live_…` from app.revenoid.com → Settings → API Keys |
| `REVENOID_API_URL` | no | `https://core.revenoid.com` | Override for self-hosted / staging / local dev (`http://localhost:8000`) |

## Security

- API keys are sha256-hashed at rest on Revenoid's side; the secret value lives only in your env
- Revoke a leaked key from Settings → API Keys (next request 401s instantly)
- Treat `rvk_live_*` like a password — anyone with it can spend your credits
- The plugin never reads or writes anything outside its install directory and the configured API endpoint

## Support

- Docs: [revenoid.com/mcp](https://revenoid.com/mcp)
- Issues: [github.com/Revenoid-Inc/revenoid-claude-plugin/issues](https://github.com/Revenoid-Inc/revenoid-claude-plugin/issues)
- Email: [support@revenoid.com](mailto:support@revenoid.com)

## License

MIT
