---
description: Generate a pre-call briefing for an upcoming meeting. Use when the user asks to "prep for", "brief me on", "get ready for", or "research before" a meeting / call / demo. Pulls calendar events to find the meeting, then chains research_account → find_person → search_call_transcripts → lookup_linkedin_posts → generate_message(callprep) for a complete brief.
---

# Pre-call prep

The user has a meeting coming up and wants a brief before it. Build a comprehensive prep doc using calendar context + every signal we have on the company and the people on the call.

The user's input ($ARGUMENTS) may be:
- A company name ("prep me for the Acme call")
- A person's name ("brief me on Stephanie before tomorrow's call")
- A vague reference ("my next call", "tomorrow's demo") — use the calendar to disambiguate

## Step 1 — Find the meeting

Call **`get_calendar_events`** with `date: "today"` (or "tomorrow" / specific date if the user said when). This returns business meetings filtered to those with external attendees.

If the user named a specific company or person, find the matching event. If they said "my next call", pick the soonest upcoming one. If multiple match, ask the user which one.

Capture:
- Meeting title + time
- External attendee emails + names
- Their email domain (= target company domain)

## Step 2 — Research the company

Call **`research_account`** with `companyName` derived from the attendee domain (use `lookup_company` first if you only have a domain).

Pull score, matched signals, and top insights. Flag the strongest 1–2 talking points.

## Step 3 — Get history with this account

Call **`search_call_transcripts`** with the company's LinkedIn URL or name.

If we've talked to them before:
- Most recent call date + summary
- Who from our side and theirs was on it
- Any open action items / commitments

If no prior calls, note "First touch with this account."

## Step 4 — Profile each external attendee

For each unique attendee from step 1:

a. Call **`find_person`** with `name` + `domain` to get their LinkedIn URL + title (if you don't have it from the calendar already).

b. Call **`lookup_linkedin_posts`** with their `linkedinUrl` (last 30 days, max 3 posts) to surface fresh talking points — recent announcements, hires, promotions, conference talks.

Keep this LIGHTWEIGHT — for each attendee:
- Name + title + tenure
- 1 fresh signal from their LinkedIn (only if there's something genuinely useful — skip if their feed is dormant)

## Step 5 — Generate the structured brief

Call **`generate_message`** with:
```
messageType: "callprep"
userQuery: "Pre-call brief for <meeting title> on <date>. Company: <name>, score <X>, signals: <top 2>. Attendees: <list>. Past engagement: <step 3 summary or 'first touch'>."
```

Pass any `messagingAgentId` if `list_messaging_agents({ messageType: "callprep" })` returned a sticky default for the user.

This is sync (`callprep` is not in the heavy-types list), so you'll get the brief back inline.

## Step 6 — Present the brief

Format for the user:

> ### 📅 [Meeting title] — [time]
>
> **Account:** [name] · Score [X]/A · [#] employees · [tech stack]
> **Attendees:** [each, with title and 1 fresh signal]
> **History:** [last call summary OR "first touch"]
> **Recommended angle:** [the strongest matched signal as a hook]
> **Open questions to ask:** [3 questions tied to signals]
>
> *[Inline content from generate_message]*

End with a **"What's next?"** section containing EXACTLY 3 backtick-quoted prompts on their own lines. Backtick-quoted prompts render as tab-acceptable suggestions in Claude Code — each MUST be a complete, ready-to-send prompt. Pick the three best follow-ups given the brief you just produced. Example shape (adapt content to actual meeting):

## What's next?

- `Draft a post-call follow-up email template using my callprep agent`
- `Search past calls for any open commitments from <Company>`
- `Pull more LinkedIn posts from <attendee> for warm-up material`

## Notes for the model

- Don't enrich attendee phone numbers / personal emails — overkill for prep.
- `lookup_linkedin_posts` is paid per call — cap to 3 attendees max, even if more are on the meeting.
- If the user has no calendar connected (`get_calendar_events` returns nothing), fall back to asking them to name the company + the people, then run from step 2.
- Skip step 4b (LinkedIn posts) entirely if Coresignal isn't returning useful posts for this profile (low engagement, dormant feed).

## Error handling

If any tool call returns an error, recognize the pattern and route the user to the right recovery — DON'T just surface a raw error message and stop:

| Error contains… | Tell the user |
|---|---|
| `INSUFFICIENT_CREDITS` / 402 | "You've used your free credits. Upgrade at https://app.revenoid.com/p2p/pricing or run `/revenoid:setup` for the full breakdown." |
| `INVALID_API_KEY` / 401 / `revoked` | "Your API key isn't working. Mint a new one at https://app.revenoid.com/p2p/settings and update `REVENOID_API_KEY`. Run `/revenoid:setup` for steps." |
| `API key required` | "Run `/revenoid:setup` — the plugin can't see your API key env var." |

Always end with a tab-acceptable recovery prompt like `/revenoid:setup` or `Open https://app.revenoid.com/p2p/pricing`.
