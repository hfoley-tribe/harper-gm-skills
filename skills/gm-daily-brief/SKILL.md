---
name: gm-daily-brief
description: Generate today's daily brief for Harper Foley (GM at Tribe AI). Pulls Google Calendar, Slack priority channels, Gmail VIP threads, recent Sybill calls, recent Granola notes, every active engagement README's open commitments, and the KKR Project OS memory directory if present, then writes a structured brief to `40-Briefs/daily/YYYY-MM-DD.md`. Use when Harper says "morning brief", "daily brief", "what's on my plate today", "what does today look like", "standup prep", or invokes `/brief`. Do not use for end-of-day capture, weekly retrospectives, or single-engagement pulse checks.
---

# GM Daily Brief

What Harper reads with coffee. The brief tells him what's on his plate today, in priority order, with the context he needs for each meeting and the open threads he owes.

A good brief is **scannable in 60 seconds with clear actions**. A bad one is a wall of synthesized text he has to re-read. When in tension, cut.

## When to use

- Harper opens Claude Code in the morning and asks for the daily brief
- Phrases that should trigger: "morning brief", "daily brief", "what's on my plate today", "what does today look like", "standup prep", `/brief`
- Re-runs mid-morning if calendar shifts substantially or new high-priority signal arrives (replaces today's file)

## When NOT to use

- Single-engagement status check before a meeting → use `engagement-pulse` (when built)
- End-of-day capture / interview → use `gm-evening-debrief` (when built)
- Weekly retrospective → use `gm-weekly-rollup` (when built)
- Cadence-specific prep (Sun / Mon / Tue / Fri) → use `gm-cadence-prep --mode {day}` (when built)
- "What did we discuss with {client}?" — direct Sybill / Granola lookup, not a brief

## Inputs

- **Today's date** — default to local today, America/New_York. If invoked after 11pm ET, ask whether to brief for today still or for tomorrow.
- **"Since" timestamp for incremental pulls** — default to `generated_at` from yesterday's brief at `40-Briefs/daily/{yesterday}.md`; fallback to "24 hours ago" if no prior brief exists.
- **Existing brief at today's path** — if `40-Briefs/daily/YYYY-MM-DD.md` already exists, ask Harper: overwrite, append a re-run section, or write to `YYYY-MM-DD-rerun-HHMM.md`.

## Steps

Numbered. Each step says what data to pull, what tool category to use (look up the specific tool with `tool_search` — don't hardcode names), and what to write where. If a step fails, do not silently skip — record the failure status for the Sources footer.

### 1. Establish "since" timestamp + check for existing brief

- Read `40-Briefs/daily/{yesterday}.md` if it exists; capture its `generated_at` value.
- Read `40-Briefs/daily/{yesterday}-evening.md` if it exists; if so, parse its `tomorrow_priorities:` YAML block and hold it in memory — these are Harper's own next-day priorities and will anchor the "Suggested priorities" section.
- Check whether `40-Briefs/daily/{today}.md` exists. If yes, ask Harper how to handle (overwrite / re-run-suffix file / append).
- Set the pull window: `[since, now]` where `since = yesterday brief's generated_at` if present, else `now - 24h`.

### 2. Calendar scan — today + tomorrow morning

- Pull today's events from Google Calendar (look up the calendar tool via `tool_search "calendar"`).
- Also pull tomorrow before 10am ET (so Harper sees an early Tuesday/Friday meeting in Monday's brief).
- For each event, classify which engagement it touches by matching: attendee email domains (`@hp.com`, `@kkr.com`, etc.), title keywords, calendar description.
- Note OOO blocks, focus blocks, personal blocks separately — these affect "Today's shape".

### 3. For each touched engagement, load context

- Read `10-Engagements/{code}/CLAUDE.md` (client-specific judgment) and `10-Engagements/{code}/README.md` (current state).
- **If Pattern A** (check by `pattern: A` in README frontmatter; today only `kkr-insurance`):
  - Try to read the Project OS memory files. **For KKR specifically, the path is `~/Projects/kkr-insurance/kkr-insurance-project-os/memory/`** — read `project-state.md`, the most recent `daily/*.md`, and `action-items.md`. (Other future Pattern A engagements may use a different layout — check the engagement README's `project_os_path` frontmatter field.)
  - If the path doesn't exist or `daily/` is empty (e.g., Project OS scaffolded but unpopulated — currently the case for KKR), surface what's there and flag the rest in the Sources footer (`⚠️ Project OS kkr-insurance: scaffolded, no daily logs yet`). Do not error.
  - Check `config/project.json` for the `visibility` flag. If `tribe-shared` (KKR is `tribe-shared`), **never surface financials** in the brief.
- **If Pattern B**: read the most recent file in `10-Engagements/{code}/meetings/` if any, plus `10-Engagements/{code}/notes/` if relevant.

### 4. Slack priority channels

- Reference the canonical priority channel list in `.claude/references/data-surfaces.md` under "Priority channels for daily briefs."
- For each channel: pull unread mentions and threads with activity in the `[since, now]` window.
- Look up Slack search/read tools via `tool_search "slack"`.
- For threads needing Harper's response, capture: who's waiting, since when, link to thread, one-line context.

### 5. Gmail VIP scan

- Pull threads in `[since, now]` from VIP senders (Brandon Bright, Tom Lee, Trevor Douglass, Hadley Riegel, Victor Miller, plus engagement-domain senders matching today's calendar attendees).
- Also pull any thread mentioning today's engagement keywords (engagement names, client codes).
- If Google Workspace credentials are expired (a documented recurring failure), surface an explicit `⚠️ Gmail (auth expired — re-auth)` in Sources footer rather than silently shipping with no Gmail data.
- Look up Gmail tools via `tool_search "gmail"`.

### 6. Sybill recent calls

- Pull `list_conversations` for yesterday + today (look up via `tool_search "sybill"`).
- Prefer deterministic `list_conversations` + `get_conversation` over the LLM-y `ask_sybill` (latter has been flaky).
- For Pattern A engagements, Sybill transcripts also auto-ingest into the Project OS `memory/calltranscripts/sybill/` directory (for KKR: `~/Projects/kkr-insurance/kkr-insurance-project-os/memory/calltranscripts/sybill/`). Pulling the live list via the Sybill MCP is fresher; use the on-disk transcripts only when MCP is down.

### 7. Granola recent notes

- Pull `list_meetings` since the prior brief's timestamp (look up via `tool_search "granola"`).
- Filter to engagement-relevant meetings + Harper's 1:1s + GTM/sales-team meetings.

### 7a. Source Quotes Scratchpad (pre-synthesis attribution gate)

Before the "Suggested priorities" or any engagement-pulse synthesis lands in the brief, build an internal scratchpad of verbatim source quotes for every claim that names a person, attributes a quote, asserts call ownership, or references a product. Daily briefs are read-only for Harper, but synthesized claims here propagate into the EOW and into engagement READMEs — attribution errors at this layer compound downstream.

For each candidate claim worth surfacing, capture:

- **Source** (Sybill conversation ID / Granola meeting ID / Slack channel + ts / Gmail thread ID / vault file path)
- **Speaker** (verbatim, as the source spells the name — do not normalize from memory)
- **Timestamp** (HH:MM or message ts)
- **Verbatim quote** (the exact words — never paraphrase at this step)

Verification rules:

- Tribe-internal names: cross-check against Slack search if any doubt (Brandon Bright, Tom Lee, Trevor Douglass, Hadley Riegel, Victor Miller, Noah Gale, Jaclyn Rice Nelson).
- Tribe products: only **SkillForge, SkillHub, Claude Coach** are Tribe products (per root `CLAUDE.md` § Product Source-of-Truth). Any other product name traces to a customer product or is wrong.
- Client-side stakeholder names: verify against the engagement README + the most recent Sybill/Granola transcript that mentions them. Never write a client name from memory — pull it.

**If a claim cannot be sourced to a verbatim quote**, drop it, soften to non-attributed framing, or surface as `⚠️ Unsourced: {claim}` in the Sources footer. Do not let unsourced claims slip into the body — the daily brief feeds the EOW and the engagement READMEs.

The scratchpad lives in the drafting process only — do not persist it into the brief output. The `**Name verification**` line in the Sources footer (Step 13) is the visible audit trail.

### 8. Cross-reference open threads

- Aggregate open threads from:
  - Each engagement README's "Open Commitments / Owed Actions" section
  - `memory/action-items.md` for Pattern A engagements
  - Slack mentions awaiting Harper's response (from step 4)
  - Gmail threads awaiting Harper's response (from step 5)
- Dedupe (a Slack thread and an engagement README entry can refer to the same item).
- Split into "I owe" (Harper's response) vs. "Waiting on others" (someone else owes Harper).

### 9. Compute "no new signal" hashes

- For each source, hash the pulled content (or the count + IDs of pulled items, simpler).
- Compare against the prior brief's hashes if recorded.
- If a source's hash matches yesterday's, in the brief's Sources footer write `✅ {source}: no new signal since {prior brief timestamp}` and **do not re-render the same items** in body sections. (First run, no prior hashes — render everything.)

### 10. Synthesize "Suggested priorities"

In order:
1. Lift Harper's `tomorrow_priorities` from yesterday's evening file (step 1) verbatim, in his words. These are primary.
2. Layer live signal: if today's calendar / Slack / Gmail surfaces a higher-urgency item Harper hadn't flagged the night before, add it but make clear it's an *override candidate* with brief reasoning ("Note: GEV team posted overnight asking about Phase 2 scope — may bump above your stated #2").
3. If no evening file exists, derive priorities from: 90-day-plan priority stack (GE Vernova #1, Elliott #2, KKR #3, HP paused, C&W shadow) + today's calendar + the most recent open commitments.

### 11. Surface scenario-log triggers

- Skim `15-Onboarding/scenario-log.md` for entries whose triggers match today's signal:
  - "Sponsor pushes meeting >2 weeks out" → check whether any sponsor rescheduled >2 weeks
  - "Sponsor goes silent for >3 hours" → check VIP Gmail/Slack response gaps
  - "Bake-off objection in mid-cycle" → check today's calendar for KKR-style calls
  - etc.
- Surface matched scenarios in "Today's shape" or as a one-liner under the relevant meeting.

### 12. Write the brief

- Path: `40-Briefs/daily/{today}.md`
- Use the template at `.claude/skills/gm-daily-brief/templates/daily-brief.md`
- Fill every section. If a section is genuinely empty, write `_None._` rather than removing the section.
- Frontmatter `generated_at` must be the current local time in `HH:MM ET` format.
- Frontmatter `sources` is an array of source identifiers actually pulled (e.g., `[calendar, slack, gmail, sybill, granola, project-os, vault]`); omit any that fully failed.

### 13. Sources footer + drift callout

- Write the Sources footer per the template, with ✅ / ⚠️ / ❌ per source and one-line status (count + freshness).
- If any source is ❌ or ⚠️, add a one-line callout at the top of "Today's shape" so Harper sees it before mid-day. Example: `⚠️ Heads up: Gmail auth expired — re-auth before relying on inbox highlights.`
- Append a `**Name verification**` line listing ✅ / ⚠️ per proper noun and product reference that appears in the brief, with the verbatim source quote each was checked against (from the Step 7a scratchpad). Any ⚠️ items must explain what couldn't be confirmed.

## Outputs

- **Primary**: `40-Briefs/daily/YYYY-MM-DD.md` — the brief itself.
- **Side effects**: none. The skill is read-only against engagement READMEs, the 90-day-plan, and the scenario-log. Amendments to those files are the evening-debrief skill's job, not this one.

## Conventions

- **Voice**: direct, terse, no hedging. *"Verona kickoff is Wednesday — read the SOW before, you haven't yet."* Not *"You may want to consider reviewing..."*. Per `40-Briefs/CLAUDE.md`.
- **Length**: scannable in 60 seconds. If the brief runs long, cut "Engagement pulse" and "Inbox highlights" first; never cut "Suggested priorities" or "Open threads I owe."
- **Honesty about uncertainty**: when data is incomplete, say so. *"Three Slack mentions in `#kkr-insurance-account` but I couldn't pull thread context — check manually"* beats fabricated synthesis.
- **Pattern A financials**: never surface contract values, rates, or margins from `tribe-shared` Project OS repos. KKR is `tribe-shared` until proven otherwise.
- **Tool name churn**: do not hardcode tool IDs in the prompt. Use capability descriptions and `tool_search`. Per `.claude/skills/CLAUDE.md`.
- **Scenario-log surfacing**: surface scenarios as one-liners adjacent to the relevant meeting/thread, not as a separate top-level section. Avoid scenario-log noise drowning the brief.
- **Carry-forward**: if Harper's evening priorities from yesterday weren't completed and remain valid, they carry forward — but mark them with `(carried from {date})` so Harper notices stagnation.

## Failure modes

- **iCloud sync lag**: just-written files occasionally aren't immediately readable. Don't loop on read-after-write; trust the write succeeded if Write returned without error.
- **MCP timeout**: if a single source times out, mark it ⚠️ in Sources footer with the error, continue with other sources, write a partial brief. Better than no brief.
- **Empty calendar day**: still write the brief. "Today's shape" notes the empty day; "Suggested priorities" leans on yesterday's evening priorities + open commitments.
- **No prior brief**: first-run mode. Use a 24h fallback window. No carry-forward priorities possible. Note this in Sources footer.
- **Engagement folder missing or malformed README**: skip that engagement's pulse line, note it in Sources footer (`⚠️ Vault: {code} README missing or unparseable`).
- **Attribution can't be sourced**: if a name, quote, product reference, or call-ownership claim has no verbatim source in the Step 7a scratchpad, drop the line or surface `⚠️ Unsourced: {claim}` in the Sources footer. Never synthesize a name or quote that no source supports — daily-brief errors propagate downstream into the EOW and engagement READMEs.

## Verification (Phase 1 dogfood)

Per the implementation plan: validate over **5 consecutive workdays** before building Phase 2 skills. Each morning Harper:

1. Runs `/brief`.
2. Reads — does it take ≤ 60 seconds? Does he feel oriented?
3. Spot-checks one section: is the engagement pulse line for today's most-active engagement actually accurate?
4. Reports back what was wrong / missing / noisy.

Tune description, source ordering, freshness logic between runs.
