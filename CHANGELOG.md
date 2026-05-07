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

[Unreleased]: https://github.com/Revenoid-Inc/revenoid-claude-plugin/compare/v0.1.0...HEAD
[0.1.0]: https://github.com/Revenoid-Inc/revenoid-claude-plugin/releases/tag/v0.1.0
