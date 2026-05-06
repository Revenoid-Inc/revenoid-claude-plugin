# Revenoid — AI Sales Workflow for Claude Code

> **Discover, research, enrich, and engage prospects through 23 unified tools** spanning Salesforce, HubSpot, Google Calendar, Microsoft 365, Gong, Chorus, Avoma, Coresignal, Apollo, and Fiber AI — with built-in Messaging Agents that draft outreach in your voice.

**Free plan ships with 500 credits.** No credit card required.

---

## Quick start (3 minutes)

### 1. Install

```bash
claude plugin install revenoid
```

### 2. Get an API key

1. Sign up at **[admin.revenoid.com/p2p/signup](https://admin.revenoid.com/p2p/signup)** (Google / Microsoft SSO, or email).
2. Open **Settings → API Keys** at [admin.revenoid.com/p2p/settings](https://admin.revenoid.com/p2p/settings).
3. Click **New key**, name it (e.g. "Claude Code"), copy the value — it shows **only once**.

### 3. Set it as an env var

**zsh** (macOS default):
```bash
echo 'export REVENOID_API_KEY="rvk_live_..."' >> ~/.zshrc
source ~/.zshrc
```

**bash**:
```bash
echo 'export REVENOID_API_KEY="rvk_live_..."' >> ~/.bashrc
source ~/.bashrc
```

**fish**:
```bash
set -Ux REVENOID_API_KEY "rvk_live_..."
```

**Windows PowerShell**:
```powershell
[Environment]::SetEnvironmentVariable("REVENOID_API_KEY", "rvk_live_...", "User")
```

### 4. Verify

Restart Claude Code (`/exit` then `claude` again — env vars only load on launch). Then run:

```
/revenoid:setup
```

You should see ✅ "You're connected to Revenoid" with your account profile. If not, the setup skill self-diagnoses and walks you through whatever's wrong.

> **Heads-up about updates:** When the plugin's MCP server adds new tools (we ship roughly weekly), `/reload-plugins` is enough for skill updates but **not** for new tools. Fully restart Claude Code (`/exit` then `claude`) so it reconnects the MCP server and picks up the latest tool list. Check `/revenoid:setup` if anything looks stale.

---

## What you can do

The plugin ships with **9 task-oriented skills** that Claude Code invokes automatically based on what you describe:

| Say something like… | Skill that fires | What happens |
|---|---|---|
| *"Prospect Anthropic for me"* | `/revenoid:prospect-account` | Resolves the company → scores it → finds prospects → enriches with verified emails → returns ranked contact list |
| *"Brief me on my next call"* | `/revenoid:pre-call-prep` | Pulls the meeting from your calendar → researches the company + attendees → searches past Gong/Chorus calls → generates a callprep brief |
| *"Find me 10 cybersecurity SaaS companies"* | `/revenoid:find-and-engage` | Discovers net-new accounts based on your saved ICP → optionally finds prospects + drafts outreach |
| *"Build an account plan for OpenAI"* | `/revenoid:account-plan` | Researches the account → kicks off async accountplan generation via your trained agent → polls until done (3–7 min) |
| *"Enrich the top 5 prospects you found"* | `/revenoid:enrich-and-sync` | Adds verified work email (+ optionally phone, personal email) → checks against your CRM for duplicates |

Plus **utility commands** for direct control:

| Command | What it does |
|---|---|
| `/revenoid:help` | Catalogue of example prompts. Try `/revenoid:help tools` for the full 23-tool list. |
| `/revenoid:setup` | First-run + diagnostics (run this if anything seems off) |
| `/revenoid:credits` | Credit balance + plan info |
| `/revenoid:agents [type]` | List your saved messaging agents (filter by type: email, callprep, etc.) |
| `/revenoid:icp [name]` | Show / switch your active ICP setting |

You can also call any of the 23 underlying MCP tools directly if you want fine-grained control.

---

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

---

## Pricing

All 23 tools share your Revenoid credit pool — same as the in-app workspace.

| Tool category | Cost |
|---|---|
| `get_company_info`, `search_call_transcripts`, `crm_query`, `get_calendar_events`, all `list_*` tools | **Free** |
| Fiber/Apollo lookups, person/company lookups, account research | Per call |
| `enrich_contacts` | Per channel (work email / personal / phone) |
| Coresignal calls (LinkedIn posts, job postings, career-history search) | Per call |
| `generate_message` (any type) | Per call |

**Free tier**: 500 credits — enough for ~50 enrichments or ~10 account plans to evaluate.

**Paid plans** start at $20/mo. Manage at [admin.revenoid.com/p2p/billing](https://admin.revenoid.com/p2p/billing).

---

## Troubleshooting

### "No revenoid tools available"
Plugin's MCP server didn't start. Most likely the env var isn't set. Run `/revenoid:setup` — it will diagnose and walk you through.

### "Invalid API key" / 401
Key was revoked or rotated. Mint a new one at [admin.revenoid.com/p2p/settings](https://admin.revenoid.com/p2p/settings) and update your env var. `/revenoid:setup` covers the steps.

### "Insufficient credits"
You've used your monthly allotment. Either wait for next month's reset, or upgrade at [admin.revenoid.com/p2p/billing](https://admin.revenoid.com/p2p/billing).

### Everything else
Run `/revenoid:setup` — it self-diagnoses. If still stuck, [open an issue](https://github.com/Revenoid-Inc/revenoid-claude-plugin/issues) with the error message and your `claude --version`.

---

## How it works

```
Claude Code  →  npx @revenoid/mcp-server (stdio)  →  HTTPS  →  core.revenoid.com/api/v2/mcp  →  23 tools
```

The plugin runs a tiny stdio proxy locally that forwards every JSON-RPC tool call to Revenoid's MCP endpoint over HTTPS, attaching your API key. **No data leaves your machine except through that one authenticated channel.**

## Configuration

| Env var | Required | Default | Notes |
|---|---|---|---|
| `REVENOID_API_KEY` | yes | — | `rvk_live_…` from admin.revenoid.com → Settings → API Keys |
| `REVENOID_API_URL` | no | `https://core.revenoid.com` | Override for self-hosted / staging / local dev |

## Security

- API keys are sha256-hashed at rest on Revenoid's side; the secret value lives only in your env
- Revoke a leaked key from Settings → API Keys (next request 401s instantly)
- Treat `rvk_live_*` like a password — anyone with it can spend your credits
- The plugin never reads or writes anything outside its install directory and the configured API endpoint

## Support

- 📚 Docs: [revenoid.com/mcp](https://revenoid.com/mcp)
- 🐛 Issues: [github.com/Revenoid-Inc/revenoid-claude-plugin/issues](https://github.com/Revenoid-Inc/revenoid-claude-plugin/issues)
- 📧 Email: [support@revenoid.com](mailto:support@revenoid.com)

## License

MIT
