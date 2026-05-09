# Changelog

All notable changes to the Revenoid Claude Code plugin are tracked here.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

> **Note:** the plugin is a thin wrapper that pins skills + slash commands and
> points at `@revenoid/mcp-server` (npm) which proxies to `core.revenoid.com`.
> Backend tool changes (new tools, fixes to existing tools) ship through the
> backend deploy and propagate to all clients without a plugin release. Only
> changes to skills, slash commands, README, or the npm proxy version pin
> warrant a plugin version bump.

## [Unreleased]

## [0.1.3] — 2026-05-09

### Changed
- **Skill descriptions broadened with verb phrasings.** Closes the routing
  race we hit on Cowork against Anthropic-provided skills. Real failure
  case: user typed *"write a pre-meeting prep for Harness"* and Cowork
  routed to `anthropic-skills:next-call-guide` instead of our
  `revenoid:pre-call-prep` because our description listed phrasings like
  "prep for", "brief me on", "build a brief on" but didn't include
  the verb **"write"**. Same gap for `account-plan` (against
  `anthropic-skills:strategic-opportunity-summary`) and `find-and-engage`
  (against the `anthropic-skills:apollo-*-prospector` skills).

  Added phrasings cover the full verb space — write, create, build,
  generate, draft, make, prepare, produce — × the noun space:
    pre-call-prep:    prep, brief, callprep, plan, guide, doc, deck
    account-plan:     plan, brief, summary, deck, doc, playbook
    find-and-engage:  list, batch, leads, prospects, target accounts

  Each description now explicitly names the competing native skills it
  should win against AND why (no access to user's voice / CRM /
  transcripts / ICP). Skill router has more reasons to pick us when the
  user is genuinely doing sales work.

### Notes on the broader pattern
- Skill-description routing is whack-a-mole. Every quarter Anthropic
  ships another skill that competes. The structural fix is the
  `revenoid_workflow` tool (Phase 11 backend, ships separately) — tool
  descriptions don't compete for routing the way skill descriptions do.
  This release patches the immediate failure modes; the orchestrator is
  the long-term answer.

## [0.1.2] — 2026-05-08

### Added
- **`messaging-policy` skill** — universal Revenoid policy for ALL sales-content
  generation (emails, LinkedIn, sequences, callprep, account plans, briefs, posts,
  presentations). Establishes the three-tier hierarchy:
    1. `list_messaging_agents` → `generate_message` with `messagingAgentId`
       (user's trained agent — preferred path)
    2. `generate_message` without `messagingAgentId` (framework engine still
       uses the user's company voice — fallback)
    3. Claude prose composition (last resort, only after both tiers error,
       and only with explicit user disclosure)
  Auto-fires on broad generation phrasing including "show me the premeeting
  plan for X", "build a brief on Y", "draft outreach to Z", etc. Closes the
  gap that let Cowork's native PDF skill win against `pre-call-prep` for
  prompts like "show me premeeting plan for Spectro Cloud".

### Changed
- **`pre-call-prep` skill** — added a top-of-file ANTI-PATTERN ALERT calling
  out the most common failure mode (composing the brief from gathered
  context instead of via `generate_message(callprep)`). Step 5 is now
  marked **MANDATORY** with explicit TIER 1 / TIER 2 / TIER 3 sub-steps.
  Description broadened to match phrasings like "show me the premeeting
  plan for", "give me prep for", "build me a callprep for".
- **`account-plan` skill** — added ANTI-PATTERN ALERT, Step 2 explicitly
  handles the no-agent-saved case (TIER 2 fallback) instead of failing,
  Step 3 marked **MANDATORY**. Description broadened.
- **`find-and-engage` skill** — Step 4 (cold-email drafting) now follows
  the messaging-policy hierarchy explicitly. TIER 1/2/3 fallback wiring,
  with disclosure language for TIER 3.

### Backend (ships independently via core.revenoid.com — no client release needed)
- **`generate_message` tool description** — rewritten to lead with "PREFERRED
  tool for ALL sales-context content generation" and the three-tier hierarchy.
  Visible to Claude in every session where the toolset is loaded, regardless
  of which skill (if any) fires. Most reliable lever for the cross-cutting
  policy.

## [0.1.1] — 2026-05-07

### Added
- **`crm_push_contacts`** — batch upsert contacts to Salesforce or HubSpot
  (auto-detected). Per-contact error isolation so one bad row doesn't fail
  the batch; `dryRun: true` for preview-without-write. Closes the
  read-only-CRM gap — `crm_query` could read but nothing could write.
- **`get_lead_list`** — poll a Lead Gen list by `rawListId`. Companion to
  the already-async `find_accounts_by_signals` so LLMs can wait for and
  surface results in-chat instead of dropping the user at the dashboard.
- Tool count: 24 → **26**.

### Fixed
- **`find_accounts`** no longer returns Fiber's "trending companies"
  default (NASA / Disney / Goldman / etc.) when ICP industries silently
  drop. Three-part fix: (1) `&` ↔ `and` alias normalisation, (2)
  explicit aliases for "Staffing & Recruiting" / "Logistics & Supply
  Chain", (3) refuse-to-call guard when no subject filter survives —
  now errors clearly with `FIBER_NO_SUBJECT_FILTER` instead of silent
  garbage. Response now includes `meta.filterDiagnostics` showing
  applied vs dropped filters per axis.
- **`find_accounts_by_signals`** response shape rewritten so async-job
  semantics are obvious to LLMs (was `preview: []` which was routinely
  treated as "tool returned nothing"). New shape leads with
  `job: { status: 'running', etaMinutes, pollWith: 'get_lead_list' }`
  plus a structured `next_action` block with explicit polling
  instructions.

## [0.1.0] — 2026-05-06

Initial public release.

### Added
- 9 task-oriented skills that auto-invoke based on user intent:
  - `/revenoid:setup` — first-run onboarding + connection diagnostics
  - `/revenoid:help` — catalogue of example prompts (with `tools` arg for full tool list)
  - `/revenoid:credits` — credit balance + plan info, live via `get_credits`
  - `/revenoid:agents [type]` — list trained messaging agents
  - `/revenoid:icp [name]` — show / switch active ICP
  - `/revenoid:prospect-account` — discover → score → enrich workflow
  - `/revenoid:pre-call-prep` — pulls calendar event → researches account + attendees
  - `/revenoid:find-and-engage` — discover net-new accounts matching saved ICP
  - `/revenoid:account-plan` — async account plan generation via trained agent
  - `/revenoid:enrich-and-sync` — verified emails / phones with CRM dedupe
- MCP wiring to `@revenoid/mcp-server@0.1.1` exposing 24 tools across:
  - Discover & qualify (7), Lookups (4), Enrichment & signals (4),
    Calendar (2), CRM (1), Outreach (2), Resources (2), Async polling (1),
    plus the free `get_company_info` baseline tool.
- `/revenoid:setup` self-diagnoses 4 failure modes (no env var, invalid key,
  out of credits, all-good) and walks the user through the appropriate fix.
- "What's next?" prompt suggestions appended to most skill outputs (one-tab
  acceptance to chain into the next workflow).

[Unreleased]: https://github.com/Revenoid-Inc/revenoid-claude-plugin/compare/v0.1.1...HEAD
[0.1.1]: https://github.com/Revenoid-Inc/revenoid-claude-plugin/releases/tag/v0.1.1
[0.1.0]: https://github.com/Revenoid-Inc/revenoid-claude-plugin/releases/tag/v0.1.0
