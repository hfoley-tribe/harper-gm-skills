---
name: gm-cadence-prep
description: Prep Harper for his standing weekly cadences. Four modes: `sunday` (week-ahead synthesis + Monday Slack DM), `monday` (GTM Weekly prep + personal weekly priorities note + Slack DM), `tuesday` (Bi-Weekly Team Pipeline Review prep with MEDDPICC walk + delta), `friday` (orchestrator wrapping `gm-eow-email` + `tribe-account-eow-update` per active engagement, with a checklist Slack DM). Use when Harper says "sunday prep", "monday prep", "gtm prep", "tuesday prep", "forecast prep", "friday wrap", "wrap the week", or invokes `/cadence-prep {mode}`.
---

# GM Cadence Prep

Parameterized prep skill for Harper's standing weekly cadences. Single skill, four modes (`sunday`, `monday`, `tuesday`, `friday`), each producing a different lens on the week.

## Modes

| Mode | Status | Output | Trigger |
|---|---|---|---|
| `sunday` | **IMPLEMENTED** | `40-Briefs/weekly/YYYY-Www-sunday-prep.md` (full week-ahead, vault) + Slack DM to Harper (Monday-only top-3 per account + top-3 across workstreams) | "sunday prep", "week ahead", "plan the week", `/cadence-prep sunday` |
| `monday` | **IMPLEMENTED** | `40-Briefs/weekly/YYYY-Www-monday-gtm.md` (GTM Weekly prep: pipeline snapshot + forecast + key deals + support needed) + Slack DM to Harper (personal weekly priorities note + GTM Weekly talking points + watching this week) | "monday prep", "gtm prep", "weekly priorities", `/cadence-prep monday` |
| `tuesday` | **IMPLEMENTED** | `40-Briefs/weekly/YYYY-Www-tuesday-forecast.md` (Bi-Weekly Pipeline Review prep: hygiene + MEDDPICC walk per active opp with delta vs last snapshot + EB-access red flags + #1 next-action) + Slack DM | "tuesday prep", "forecast prep", "pipeline review prep", "meddpicc walk", `/cadence-prep tuesday` |
| `friday` | **IMPLEMENTED** | Orchestrator: SFDC hygiene check, then invokes `tribe-account-eow-update` per active engagement, then `gm-eow-email`. Produces a Slack DM checklist tracking ✅/⏳/❌ per step. Sub-skills write their canonical artifacts. | "friday wrap", "friday cadence", "wrap the week", `/cadence-prep friday` |

## When to use (Sunday mode)

- Sunday afternoon / evening before the upcoming work week begins
- Phrases that should trigger: "sunday prep", "week ahead", "plan the week", "what's the week look like", "/cadence-prep sunday"
- Re-run permitted if Harper updates the plan or new info lands late Sunday

## When NOT to use

- Mid-week "what's today" check → `gm-daily-brief`
- End-of-day debrief → `gm-evening-debrief` (when built)
- Weekly retrospective looking BACK → `gm-weekly-rollup` (when built)
- Friday EOW email composition without orchestration → `gm-eow-email` directly
- Single-engagement EOW Slack post without orchestration → `tribe-account-eow-update` directly

## Inputs (Sunday mode)

- **Target week** - default to the upcoming ISO week (the Monday after today). If today is Sunday 2026-05-03, target = `2026-W19` covering 2026-05-04 → 2026-05-10. If invoked outside Sunday, ask Harper to confirm: this week or next week?
- **"Last week's range"** - the ISO week ending today (or the week prior if invoked off-Sunday). Used to pull last week's daily briefs and "what slipped".
- **Existing prep file** - if `40-Briefs/weekly/{target-week}-sunday-prep.md` already exists, ask: overwrite, append a re-run section, or write to `{target-week}-sunday-prep-rerun-HHMM.md`.

## Steps (Sunday mode)

Numbered. Each step says what data to pull, what tool category to use (look up specific tools via `tool_search`), and what to write where. If a step fails, record the failure for the Sources footer - do not silently skip.

### 1. Establish target week + last week's range

- Compute target ISO week: if today is Sunday, `target = next ISO week`; else ask Harper.
- Compute last week's range: the seven days ending today (Sun-prior to Sat-this) - this is what last week's daily briefs cover.
- Check whether `40-Briefs/weekly/{target-week}-sunday-prep.md` exists; if yes, ask Harper how to handle (overwrite / re-run-suffix / append).

### 2. Read last week's daily briefs

- Glob `40-Briefs/daily/*.md` for files in last week's range.
- For each: parse "Suggested priorities" + "Open threads I owe" + any "Override candidate" notes.
- Also read last week's evening files (`*-evening.md`) if present, for what actually got done vs. promised. *(Note: evening files don't exist yet - `gm-evening-debrief` is Phase 2. Skip cleanly if absent.)*
- Compute what slipped: priorities that recurred across multiple daily briefs without resolution.

### 3. Calendar lookahead - next 7 days

- Pull events for the target week from Google Calendar (use `tool_search "calendar"`).
- Classify each event by engagement (attendee email domains, title keywords, description).
- Identify standing meetings: Brandon 1:1, Tom 1:1, GTM weekly (Mon), bi-weekly forecast (Tue, even/odd-week TBD), Friday Team Q&A, peer-GM 1:1s, account-specific weekly cadences (GEV Monday, Dhrupan Thu 1pm PT, C&W Sal sync, etc.).
- Identify high-stakes one-off meetings: e.g., Mike (Syneos) CIO call solo, KKR 5/7 1–2pm ET call, GEV Phase 2 milestones.
- Note OOO blocks, focus blocks.

### 4. Slack priority channels - last 7 days

- Reference canonical priority channel list in `.claude/references/data-surfaces.md` ("Priority channels for daily briefs" - same set applies for Sunday prep).
- Pull last 7 days of unread mentions and threads with activity in each channel.
- Capture: open threads Harper owes, threads still unanswered by counterparts, any escalation signal.

### 5. Gmail VIP scan - last 7 days

- Pull threads in last 7 days from VIPs (Brandon, Tom, Trevor, Hadley, Victor, plus engagement-domain senders).
- Plus engagement-keyword threads.
- Flag auth failure explicitly (`⚠️ Gmail (auth expired - re-auth)`); never silently ship without Gmail data.

### 6. Sybill recent calls - last 7 days

- `list_conversations` for last 7 days; prefer deterministic over `ask_sybill`.
- Capture call outcomes that affect next week: commitments made, scope changes, sponsorship signals.

### 7. Granola notes - last 7 days

- `list_meetings` since 7 days ago.
- Filter to engagement + 1:1 + GTM/sales-team meetings. Capture what Harper noted about next-week implications.

### 8. Pipeline state - Sybill `list_deals` + Airtable projects

- Pull Sybill `list_deals` for Harper's owned book (filter by account: GE Vernova, KKR Insurance, Elliott / Syneos Health, HP).
- Pull Airtable projects for the same accounts via TribeMCP `search_airtable`.
- Note any stage changes, close-date drift, amount changes since last Sunday.

### 9. Engagement state aggregation

- For each active engagement (GEV, Elliott, KKR, HP, C&W) read `10-Engagements/{code}/CLAUDE.md` + `README.md`.
- For each: extract Open Commitments / Owed Actions, scheduled meetings, decisions pending.
- For KKR (Pattern A): also try `~/Projects/kkr-insurance/kkr-insurance-project-os/memory/project-state.md` + `memory/action-items.md`. Fall back gracefully if path missing - flag in Sources footer.
- For C&W: read-only observation; no commitments to surface as Harper's (Trevor is GM).

### 10. Read 90-day plan + scenario log

- Read `15-Onboarding/90-day-plan.md` - extract the most recent priority stack and any week-by-week intended actions for the target week.
- Read `15-Onboarding/scenario-log.md` - flag scenarios whose triggers are likely to fire next week given calendar + commitments (e.g., KKR Tue 5/7 call → "bake-off objection" + "sponsorship test" scenarios; Mike CIO call → "first-meeting-with-skeptical-CIO" pattern via Elliott brief).

### 11. Synthesize "Priority stack for the week"

- Anchor to the Brandon 5/1 priority order from `15-Onboarding/90-day-plan.md`: GEV #1 (commercial expansion), Elliott / Syneos #2, KKR Insurance #3, HP paused (no cycles unless alt-fit signal), C&W shadow only.
- Layer in time-bound commitments: Phase 2 deck due X, KKR 5/7 call, Mike (Syneos) CIO call.
- Cap at 3–5 stack items. If more than 5 want to land, that's a signal to push something to the following week - flag explicitly.

### 12. Write the prep file

- Path: `40-Briefs/weekly/{target-week}-sunday-prep.md` (e.g., `2026-W19-sunday-prep.md`)
- Use template at `.claude/skills/gm-cadence-prep/templates/sunday.md`
- Fill every section. If a section is genuinely empty, write `_None._` rather than removing.
- Frontmatter `mode: sunday-prep`, `week: {target-week}`, `range: {YYYY-MM-DD to YYYY-MM-DD}`, `generated_at: HH:MM ET`, `sources: [...]`.

### 13. Sources footer + drift callout

- Per the template: ✅ / ⚠️ / ❌ per source with one-line status (count + freshness).
- If any ❌ or ⚠️, lead "The week's shape" with a one-line heads-up.

### 14. Synthesize and send the Slack DM (Monday-only companion)

Companion artifact to the vault file. Harper reads this on his phone Sunday evening; the vault file is the deeper read for Monday morning. Same data pull, different lens.

**Target focus.** Tomorrow only (Monday by default). If invoked off-Sunday, target the next workday and call it out in the day-shape line. Do not summarize the full week here - that's the vault file's job.

**Selection rules.**

1. **Active engagements only.** Iterate `10-Engagements/{code}/README.md` where `status` is not `paused` or `shadow`. Today: GEV, Syneos, KKR. Skip HP (paused) and C&W (shadow - Trevor's deal). If a paused/shadow account has a meeting on the target day, surface it in the day-shape line only; no top-3 section.
2. **Top 3 per active engagement.** Source from each engagement's Open Commitments / Owed Actions, Pattern A `memory/action-items.md` if present, README §-specific Monday items (e.g., README §7 "Open"), and target-day calendar attendance. Cite source for every item: `(README §X)`, `(Project OS action #N)`, `(notes/path)`, or `(CLAUDE.md "{section}")`.
3. **Top 3 across workstreams, not per-workstream.** Iterate `45-Initiatives/{slug}/README.md` where `status: active`. Pick the 3 most target-day-actionable items across all workstreams (calendar meetings, EOW deadlines, time-bound asks). Workstream stubs with no target-day action don't get a slot. If fewer than 3 have actionable items, surface what you have and skip the rest. Don't pad.
4. **Time-urgency ordering.** Order engagement sections by target-day time-urgency, most urgent first (e.g., engagement with the 12pm meeting before the one with rolling reviews). Not alphabetical, not priority-stack order.
5. **Day-shape line at top.** Chronological list of target-day meetings with 1-line context for high-stakes ones. Flag high-stakes meetings newly added in the last 72 hours with ⚠️ (Sat/Sun additions often change Monday's shape). Call out meaningful absences as their own signal (e.g., "No KKR meeting on the books" when an expected meeting didn't materialize).
6. **Honest sources footer.** Mirror the vault file's footer but tighter. Name what was pulled, name what was skipped, flag the one or two skipped sources that would have changed the message if pulled.

**Format.** Use template at `.claude/skills/gm-cadence-prep/templates/sunday-slack-dm.md`. Color tags: 🟦 / 🟨 / 🟩 per active engagement, 🟪 for workstreams. Action verb + object in bold, then 1-2 sentences of context with verbatim quotes. No em dashes (per Harper's global preferences). Voice matches the vault file - direct, terse, operator-grade.

**Send.** Target = Harper's Slack DM (user_id `U0AR9GQAU06`). Use Slack send-message capability (look up via `tool_search "slack send"`). Send direct, not draft - this is a scheduled recurring artifact, not a one-off composition.

**Edge cases.**
- **Invoked off-Sunday.** Target the next workday, not Monday. Update day-shape labeling accordingly.
- **Target day is OOO** (Harper-side, per calendar). Target the next workday and call out the OOO in the day-shape line.
- **Re-run same Sunday.** Send a fresh DM (don't try to edit prior message). Harper can ignore older ones; the most recent is canonical.
- **No active workstreams.** Skip the workstreams section entirely rather than writing "_None._" - keep the DM tight.

## Outputs (Sunday mode)

- **Primary (vault):** `40-Briefs/weekly/{target-week}-sunday-prep.md` - the deep week-ahead.
- **Primary (Slack):** DM to Harper (user_id `U0AR9GQAU06`) - the Monday-focused top-3 companion.
- **Side effects:** none. Read-only against engagement READMEs, 90-day-plan, scenario-log, 45-Initiatives. Amendments to those files belong to evening debrief or weekly rollup, not this skill.

## Conventions (Sunday mode)

- **Voice**: same as daily brief - direct, terse, no hedging. *"Mike CIO call - Trevor scheduling, Hadley signed off solo. First solo customer call. Don't wing it."* Not *"You may want to prepare carefully for..."*.
- **Length**: should fit in ~2 minutes of reading on Sunday evening. Detailed enough that Monday morning Harper isn't re-discovering the week, but not so long it becomes a slog.
- **Honesty about uncertainty**: when sources fail or commitments are vague, say so. *"Elliott channel name still TBD - Slack pull may have missed early Trevor activity"* beats fabricated synthesis.
- **Pattern A financials**: never surface contract values, rates, or margins from `tribe-shared` Project OS repos. KKR is `tribe-shared` until proven otherwise.
- **Tool name churn**: do not hardcode tool IDs in the prompt. Capability descriptions + `tool_search`. Per `.claude/skills/CLAUDE.md`.
- **Carry-forward discipline**: if a priority slipped from last week and re-appears this week, mark `(carried from W{prior})`. If it slipped twice, mark `(carried 2x - consider dropping or escalating)`.
- **Account-specific meetings**: only list each meeting once. Don't double-list a Brandon 1:1 under both "standing" and "account-specific."
- **Last week's wins**: keep brief - 3 bullets max. This is for Harper's framing, not a victory lap.

## Failure modes (Sunday mode)

- **iCloud sync lag**: see `gm-daily-brief` SKILL.md.
- **MCP timeout**: mark ⚠️ in Sources footer, continue with other sources, write a partial prep file.
- **No daily briefs from last week**: first-Sunday-after-skill-build. Skip "What slipped from last week" with note `_First run - no prior daily briefs to roll up._`.
- **Engagement folder missing or malformed**: skip that engagement's section, note in Sources footer.
- **`list_deals` returns empty**: check whether engagement is Airtable-only or hasn't been added to Sybill CRM yet (KKR Insurance has no Sybill account/deal record per Phase 1 finding). Note + fall back to Airtable + engagement README.
- **Today is not Sunday but mode `sunday` was invoked**: ask Harper which week he wants prep for. Offer "the upcoming ISO week" as default.

## When to use (Monday mode)

- Early Monday morning, before the 8:30 AM ET Weekly Priorities Slack post and the 2:00 PM ET GTM Weekly with Brandon.
- Phrases that should trigger: "monday prep", "gtm prep", "weekly priorities", "deal review prep", "/cadence-prep monday".
- Re-run permitted if signal shifts between morning prep and the 2pm meeting (e.g., a Brandon DM lands at 11am that changes the support ask).
- If invoked off-Monday, ask Harper which Monday he wants prep for. Default offer: the upcoming Monday.

## Inputs (Monday mode)

- **Target Monday** - default to today if today is Monday; else ask. Compute the ISO week from the Monday.
- **Forecast-week flag** - derive from calendar (is there a Bi-Weekly Team Pipeline Review on the Tuesday of the target week?). If yes, the prep file leads with forecast submission as a hard deadline.
- **Existing prep file** - if `40-Briefs/weekly/{target-week}-monday-gtm.md` already exists, ask Harper: overwrite, append a re-run section, or write to `{target-week}-monday-gtm-rerun-HHMM.md`.
- **Prior Sunday-prep file** - if `40-Briefs/weekly/{target-week}-sunday-prep.md` exists, treat it as the highest-value source. Monday prep is largely a delta + repackaging of Sunday's synthesis. If it doesn't exist, do a full data pull instead.

## Steps (Monday mode)

Numbered. Each step says what data to pull, what tool category to use (look up specific tools via `tool_search`), and what to write where. If a step fails, record the failure for the Sources footer; do not silently skip.

### 1. Establish target Monday + ISO week

- Compute target Monday. If today is Monday, target = today. Else ask Harper.
- Derive ISO week ID (e.g., `2026-W19`).
- Check whether `40-Briefs/weekly/{target-week}-monday-gtm.md` exists; if yes, ask Harper how to handle (overwrite / re-run-suffix).

### 2. Determine forecast-week status

- Pull calendar (use `tool_search "calendar"`) for the target week and check Tuesday for a Bi-Weekly Team Pipeline Review.
- If present, this is a forecast-submission week. Pre-meeting forecast submission is due before the 2pm GTM Weekly per Harper's all-day calendar reminder.
- Set the `forecast_week` frontmatter flag accordingly.

### 3. Load prior Sunday-prep file (if present)

- Read `40-Briefs/weekly/{target-week}-sunday-prep.md`.
- Extract: priority stack, key deals already flagged, decisions due this week, deliverables owed, scenarios in play, what slipped from last week.
- This is the spine of the Monday prep. Don't re-synthesize from raw sources what Sunday already synthesized.
- If absent, flag in Sources footer and fall back to the Sunday mode's source list to compose from scratch.

### 4. Pull live signal for delta vs Sunday

- Slack priority channels (use `tool_search "slack"`) - last 24-48hr only. Capture anything that shifted since Sunday prep: Brandon DMs, account-channel activity, new escalations.
- Calendar (last 48hr changes) - any meetings added, moved, or cancelled since Sunday prep. Flag newly-added high-stakes meetings (≤72hr old) with ⚠️ in the day-shape line.
- Sybill calls (last 48hr) - any post-Sunday calls that affect the meeting. Reference canonical priority channel list in `.claude/references/data-surfaces.md`.

### 5. Pipeline state - Sybill `list_deals` + engagement READMEs

- Pull Sybill `list_deals` (use `tool_search "sybill deals"`) for Harper's owned book: GE Vernova, KKR Insurance, Syneos Health, HP.
- For each: read `10-Engagements/{code}/README.md` frontmatter for `meddpicc_tally`, `last_reviewed`, `status`, `stage`.
- Compute week-over-week delta vs Sunday prep's pipeline snapshot + last week's daily briefs.
- KKR Insurance has no Sybill account/deal record (per Phase 1 finding) - fall back to README frontmatter + Project OS `memory/project-state.md`.

### 6. Forecast math

- Goal = $3M Q2 (per `reference_weekly_email_format` memory: Brandon's standard).
- Best Case = additional on top of Commit (per same memory; do not double-count).
- Pull Commit + Best Case + Closed-Won from Sybill `list_deals` + engagement README frontmatter.
- Compute gap-to-goal: `(Commit + Closed-Won) - Goal`. State the gap explicitly in one line.

### 7. Engagement state aggregation

- For each active engagement (GEV, Syneos, KKR) read `10-Engagements/{code}/CLAUDE.md` + `README.md`.
- For KKR (Pattern A): also read `~/Projects/kkr-insurance/kkr-insurance-project-os/memory/project-state.md` + `memory/action-items.md`. Fall back gracefully if path missing.
- Extract: state heading into the meeting, key deals to surface, support asks.
- Skip HP (paused) and C&W (shadow) for top-3 sections. Surface only if a meeting on Monday's calendar warrants a day-shape mention.

### 8. Synthesize the prep file content

- **Top of mind** - 1-2 sentences pulling from this week's commitments + post-Sunday signal. Mention any 24-48hr shifts. Write as verbatim spoken draft.
- **Announcements / shout-outs** - 0-2 items only. Skip the section if nothing landed. Each item one sentence + source.
- **Pipeline snapshot** - populate the table for GEV, Syneos, KKR, HP. Add 1-sentence notes below the table for rows where delta needs context.
- **Quarter forecast** - Commit / Best Case / Closed-Won / Goal / gap-to-goal. Flag whether this is a forecast-submission week.
- **Key deals + support needed** - 2-3 deals, each with state one-liner + support ask one-liner.
- **What I'm watching** - 1-3 bullets from Sunday-prep priority stack.

### 9. Write the prep file

- Path: `40-Briefs/weekly/{target-week}-monday-gtm.md` (e.g., `2026-W19-monday-gtm.md`).
- Use template at `.claude/skills/gm-cadence-prep/templates/monday-gtm.md`.
- Fill every section. If a section is genuinely empty, write `_None._` rather than removing.
- Frontmatter: `mode: monday-gtm`, `week: {target-week}`, `range: {YYYY-MM-DD to YYYY-MM-DD}`, `generated_at: HH:MM ET`, `forecast_week: {true|false}`, `sources: [...]`.

### 10. Sources footer + drift callout

- Per the template: ✅ / ⚠️ / ❌ per source with one-line status.
- If any ❌ or ⚠️, lead "The shape of the meeting" with a one-line heads-up.

### 11. Synthesize and send the Slack DM

Companion artifact to the vault file. Harper reads this on his phone Monday morning; the vault file is the deeper read before 2pm.

**Three blocks, in order:**

1. **🎯 Harper's weekly priorities (personal note).** 3 bullets, Harper's top 3 priorities for the week. Sourced from Sunday-prep priority stack + any live signal. **Personal self-direction anchor, not a team-facing post.** No SOP today says these go to a team Slack channel; if one ever surfaces (Brandon coaching, a sales-team SOP update), hard-code the channel in this skill and add a paste-ready constraint then.
2. **GTM Weekly talking points (2pm ET).** Top-of-mind 1-sentence draft, 2-3 key deals to surface (color-tagged 🟦/🟨/🟩), 1-2 support asks.
3. **🟪 Watching this week.** 1-3 cross-cutting bullets. Optional; cut first if length runs over.

**Format.** Use template at `.claude/skills/gm-cadence-prep/templates/monday-gtm-slack-dm.md`. Color tags 🟦/🟨/🟩 per active engagement, 🟪 for workstreams. Action verb + object in bold. Cite every item. No em dashes.

**Send.** Target = Harper's Slack DM (user_id `U0AR9GQAU06`). Use Slack send-message capability (look up via `tool_search "slack send"`). Send direct, not draft.

**Timing.** Send early Monday morning, ideally before the 9:30 AM ET deal-review prep block. No hard 8:30 AM deadline since priorities are personal-note, not team-posted.

## Conventions (Monday mode)

- **Sunday prep is the spine.** Don't re-pull every source from scratch. Read Sunday prep first; pull only what's changed since.
- **Forecast-week discipline.** When the Tuesday Bi-Weekly Pipeline Review is on the calendar, forecast submission is the highest-priority pre-meeting deliverable. Surface it explicitly in "The shape of the meeting" lead.
- **Priorities block is a personal note, not a team-facing post.** Source citations stay inline (this is for Harper's self-direction; he reads the DM with context, not as a paste-ready broadcast). The 5/10 vault investigation found no SOP that names a Slack channel for weekly priorities posting. If one surfaces, update this convention and the template's block-1 framing.
- **Best Case = additional on top of Commit** (per `reference_weekly_email_format` memory). Never roll Best Case into Commit; never double-count.
- **Pattern A financials guardrail.** KKR is `tribe-shared`; never surface contract values, rates, or margins from Project OS in the Slack DM or the vault file.
- **Tom OOO weeks.** Check Tom's calendar status; if OOO, note in the day-shape line and skip any "Tom said X this week" items unless sourced from prior week.
- **Length.** Vault file should be readable in 5 minutes. Slack DM in 60 seconds on a phone.

## Failure modes (Monday mode)

- **No prior Sunday prep file.** Fall back to full Sunday-mode data pull. Note in Sources footer. Expect 2-3x runtime.
- **Forecast-week status ambiguous.** Look for the Bi-Weekly Team Pipeline Review on Tuesday's calendar. If absent, default to non-forecast-week and flag in Sources footer.
- **Sybill `list_deals` returns stale data.** Cross-check against engagement README frontmatter (`stage`, `meddpicc_tally`, `last_reviewed`). Flag mismatch in Sources footer.

## Outputs (Monday mode)

- **Primary (vault):** `40-Briefs/weekly/{target-week}-monday-gtm.md` - the pre-meeting reference.
- **Primary (Slack):** DM to Harper (user_id `U0AR9GQAU06`) - phone-readable companion with paste-ready 8:30 block.
- **Side effects:** none. Read-only against engagement READMEs, Project OS memory, prior Sunday-prep file.

## When to use (Tuesday mode)

- Late Monday afternoon, evening, or Tuesday morning before the 4-5pm ET Bi-Weekly Team Pipeline Review (Tom Lee organizes; Brandon, Will, Orges, Tommy optional, Harper attends).
- Phrases that should trigger: "tuesday prep", "forecast prep", "pipeline review prep", "meddpicc walk", "/cadence-prep tuesday".
- Re-run permitted if MEDDPICC scoring refreshes Monday night or a new call lands Tuesday AM.

## Inputs (Tuesday mode)

- **Bi-weekly status** - confirm this week is ON the bi-weekly cadence by checking Harper's calendar for the Tuesday 4-5pm ET event. If OFF, write the prep anyway (forecast hygiene) and flag in the frontmatter + opening paragraph that there's no meeting.
- **Target meeting date** - the upcoming Tuesday (or today if invoked Tuesday AM).
- **ISO week** - the week containing the Tuesday meeting.
- **Existing prep file** - if `40-Briefs/weekly/{week}-tuesday-forecast.md` already exists, ask Harper: overwrite, append a re-run section, or write to `{week}-tuesday-forecast-rerun-HHMM.md`.

## Steps (Tuesday mode)

Numbered. Each step says what data to pull, what tool category to use (look up specific tools via `tool_search`), and what to write where. Read-only against MEDDPICC files. Record source failures in the Sources footer; do not silently skip.

### 1. Confirm bi-weekly cadence status + target week

- Pull Harper's Tuesday calendar (use `tool_search "calendar"`). Look for the "Bi-Weekly Team Pipeline Review" event 4-5pm ET.
- If the event exists this week → cadence is ON. If not → cadence is OFF; flag in frontmatter `bi_weekly_status: off` and in the opening paragraph.
- Check whether `40-Briefs/weekly/{week}-tuesday-forecast.md` exists; ask how to handle if so.

### 2. Pull Sybill `list_deals` for Harper's book

- Filter to GE Vernova, Syneos Health, KKR Insurance. Skip HP (paused) and C&W (shadow).
- Capture: SFDC stage, close date, amount, last activity date, owner.
- Note any stage / close-date / amount drift since the prior Tuesday-prep file (if it exists).

### 3. Read canonical MEDDPICC scoring per active engagement

- For each of GEV, Syneos, KKR: read `10-Engagements/{code}/meddpicc.md`.
- Capture: aggregate tally (`aggregate_green / yellow / red`), per-field status + evidence, "What converts the deal forward" section.
- If a canonical `meddpicc.md` is missing for an engagement, flag in Sources (❌) and skip that engagement's field-by-field walk; surface the gap loudly in "The meeting's shape".

### 4. Read MEDDPICC snapshots for delta computation

- For each engagement: glob `10-Engagements/{code}/reference/meddpicc-snapshots/*.md` sorted by date.
- "Prior" = most recent snapshot dated *before the last bi-weekly Tuesday* (typically ~2 weeks ago). If no qualifying snapshot exists, write `_no prior snapshot - first scoring_` and skip the delta column.
- Compute per-field deltas: 🟥→🟨, 🟨→🟢, etc.

### 5. Read engagement READMEs + CLAUDE.mds

- For each engagement: `10-Engagements/{code}/CLAUDE.md` + `README.md`.
- Pull EB / "People" / "Political reality" sections to support the EB-access red flag check.
- Pull Open Commitments / Owed Actions to support the Momentum & Accountability commitment.

### 6. Pattern A - KKR Project OS pull

- Try `~/Projects/kkr-insurance/kkr-insurance-project-os/memory/project-state.md` + `memory/action-items.md`.
- If Project OS state has movement that hasn't propagated to vault `meddpicc.md` (e.g., functional-leader call landed but vault scoring is pre-call), flag in the KKR sub-section: "vault scoring lags Project OS - refresh owed".
- Fall back gracefully if path missing - flag in Sources footer.

### 7. Read prior Tuesday-prep + Sunday-prep

- Glob `40-Briefs/weekly/*-tuesday-forecast.md` for the immediately prior bi-weekly Tuesday (if any). Capture the "Harper's #1 commitment" from that prep and assess whether it shipped.
- Read `40-Briefs/weekly/{this-week}-sunday-prep.md` to pull priority stack alignment for the Momentum commitment.

### 8. Pipeline hygiene synthesis

- Build the per-engagement table (stage, close date, amount, last touch, flag).
- Flag candidates for kill / downgrade / re-stage: close date in the past, no activity 14+ days, stage doesn't match recent call evidence.
- Draft one-sentence "why will this close in the committed timeframe" defense per deal.

### 9. MEDDPICC walk per engagement

- Per engagement: render the 8-field table with current color, prior snapshot color, delta arrow, and a one-line defense / speaking script per field.
- Capture coverage self-score (strong / partial / gap) per engagement: bias conservative per scoring discipline in `10-Engagements/CLAUDE.md`.
- Surface "likely pushback in the room" per engagement with prepared responses.

### 10. EB-access red flag portfolio view

- For each engagement: name the EB, classify Harper's access (direct / via intermediary / none), name the gap and the path forward.
- KKR is the live exposure today: Neir is operational sponsor; William Karkar has two-no-show pattern; funding gate KKR-Insurance vs. KKR-parent unconfirmed.

### 11. Synthesize Harper's #1 commitment

- Cap at one named priority action for the next 2 weeks (bi-weekly window) with specific deliverable + deadline.
- Draft 2 runner-up commitments in case Tom presses.
- Source citation required: README §X / Project OS action #N / Sunday-prep priority #N.

### 12. Write the prep file

- Path: `40-Briefs/weekly/{week}-tuesday-forecast.md`
- Use template at `.claude/skills/gm-cadence-prep/templates/tuesday-forecast.md`.
- Fill every section. If a section is empty, write `_None._`. Never delete sections.
- Frontmatter: `mode: tuesday-forecast`, `week`, `meeting_date`, `bi_weekly_status`, `generated_at`, `sources`.

### 13. Sources footer

- Per template: ✅ / ⚠️ / ❌ per source. Always specify failure mode for ⚠️ / ❌ ("snapshot >2 weeks old", "auth expired", "canonical meddpicc.md missing").
- If any ❌ or ⚠️, lead "The meeting's shape" with a one-line heads-up.

### 14. Send the Slack DM

Companion artifact. Phone-readable. Lands Monday evening or Tuesday AM.

**Selection rules.**

1. **Active engagements only.** GEV, Syneos, KKR. Skip HP (paused) and C&W (shadow).
2. **Per engagement: tally + delta + top weakness + coverage self-score.** All four lines required.
3. **Color tags fixed:** 🟦 GEV / 🟨 Syneos / 🟩 KKR (not order-dependent).
4. **Order by SFDC close-date proximity** - most-urgent first.
5. **EB-access red flags:** single line per deal where this is the gap. Skip the section if none.
6. **#1 commitment:** one paragraph naming the action, deliverable, deadline, source.
7. **Sources footer:** tighter than vault but honest about gaps.

**Format.** Template at `.claude/skills/gm-cadence-prep/templates/tuesday-forecast-slack-dm.md`. No em dashes. Voice matches the vault file.

**Send.** Target = Harper's Slack DM (user_id `U0AR9GQAU06`). Direct, not draft.

## Conventions (Tuesday mode)

- **Bi-weekly cadence: confirm before assuming.** The Pipeline Review is bi-weekly. Check the Tuesday calendar; if the event is absent, the cadence is OFF this week. Generate the prep anyway for hygiene; flag it clearly.
- **Read-only against MEDDPICC.** This skill never modifies `meddpicc.md` or snapshot files. That's the `meddpicc-scorecard-update` skill's job. If scoring is stale, surface that and recommend Harper run the update skill separately.
- **Conservative scoring matches the canonical file.** Defense lines acknowledge the current color without softening. If Metrics is RED, the defense names what closes it, not a wishful upgrade. Per `10-Engagements/CLAUDE.md`: Identify Pain is GREEN only when customer-specific named pain is captured; Metrics is RED until quantified outcomes documented; EB is YELLOW until funding + named individual + sponsorship reach all confirmed; Champion is YELLOW until customer-side stakeholder actively advocating internally; Paper Process is RED until MSA + security review + regulatory cert posture mapped. When in doubt, score down.
- **Delta = current vs. last-bi-weekly snapshot.** Not "most recent snapshot" if that snapshot is mid-week; use the snapshot that pre-dates the last Tuesday meeting.
- **Active engagements only.** GEV, Syneos, KKR. HP and C&W out of scope.
- **No financials in Pattern A Project OS-touching outputs** (KKR is `tribe-shared`). Vault file is fine.

## Failure modes (Tuesday mode)

- **Bi-weekly is OFF this week.** Write the prep anyway, flag `bi_weekly_status: off` in frontmatter, lead with "this week is OFF cadence - no Tuesday meeting".
- **Canonical `meddpicc.md` missing for an engagement.** Skip that engagement's field-by-field walk; flag in Sources (❌); surface loudly in "The meeting's shape" because Harper will have no walk to give in the meeting. Recommend `meddpicc-scorecard-bootstrap` or `meddpicc-scorecard-update`.
- **No prior snapshot for delta.** Write `_no prior snapshot - first scoring_` and skip the delta column. This is expected for the first Tuesday-prep run after a fresh scoring.
- **Snapshot is older than 4 weeks.** Mark ⚠️ in Sources; the delta is stale; recommend Harper run `meddpicc-scorecard-update` before the meeting.
- **Sybill `list_deals` returns empty for an engagement.** KKR has no Sybill account record per Phase 1 finding - fall back to Airtable + engagement README. For GEV / Syneos: investigate, then flag.
- **MCP timeout.** Mark ⚠️ in Sources, continue with other sources, write a partial prep file.
- **Project OS path missing for KKR.** Flag ❌ in Sources for the Pattern A entry; skip step 6's lag check; vault `meddpicc.md` is still the canonical scoring.
- **No prior Tuesday-prep file.** First run: write `_First Tuesday-prep run. No prior commitments to track._` in the "What was committed last bi-weekly" section.
- **Today is Tuesday after 4pm.** Ask Harper: prep for the meeting that just ended (next bi-weekly's #1 commitment retrospective) or for the next bi-weekly two weeks out?

## Outputs (Tuesday mode)

- **Primary (vault):** `40-Briefs/weekly/{week}-tuesday-forecast.md` - pre-meeting reference and 20-min MEDDPICC walk script.
- **Primary (Slack):** DM to Harper (user_id `U0AR9GQAU06`) - phone-readable companion, lands evening before or morning-of.
- **Side effects:** none. Read-only against `meddpicc.md`, snapshots, READMEs, CLAUDE.mds, Project OS memory, prior brief files. MEDDPICC refreshes are out of scope (use `meddpicc-scorecard-update`).

## When to use (Friday mode)

- Friday afternoon, against the 3:00 PM ET / 3:30 PM ET self-block reminders ("Complete Weekly Deal Review", "Prep for Bi-weekly Forecast", "Send Tom End of Week Update").
- Phrases that should trigger: "friday wrap", "friday cadence", "wrap the week", "/cadence-prep friday".
- Re-runs allowed if a late-Friday meeting materially changes the picture for any engagement.

Friday mode is an **orchestrator**, not a producer. It sequences two existing skills and tracks what's done vs. owed. No new vault artifact lands from this mode itself.

## Inputs (Friday mode)

- **Today's date** - default local today, America/New_York. If invoked outside Friday, ask: "wrap for the ISO week ending today?"
- **Active engagements** - derive from `10-Engagements/{code}/README.md` frontmatter where `status: active`. Today: `ge-vernova`, `syneos-health`, `kkr-insurance`. Skip `hp` (paused) and `cushman-wakefield` (shadow: Trevor's account).
- **Sub-skill availability** - confirm both `gm-eow-email` and `tribe-account-eow-update` are discoverable before sequencing. If either is missing, stop and report to Harper rather than running a partial wrap.

## Steps (Friday mode)

Numbered. Friday mode delegates the heavy lifting; each step says which sub-skill to invoke or which hygiene check to run, and what status to record for the Slack DM checklist.

### 1. SFDC + pipeline hygiene pre-flight

For each active engagement, read `10-Engagements/{code}/README.md` (and `meddpicc.md` if present), pull the Sybill `list_deals` record for the account, and confirm: stage, close date, amount, MEDDPICC tally are current vs. the most recent material event. The Sunday-evening forecast submission relies on this data being right.

Record one of three per engagement:
- ✅ all four fields current
- ⏳ minor staleness Harper can fix in 2 minutes (e.g., close date drift by a week)
- ❌ material staleness that would mislead Tom or Brandon (stage mismatch, amount drift >25%)

Surface every ⏳/❌ in the SFDC hygiene flags section of the Slack DM. Do not run the downstream skills if anything is ❌: Harper fixes SFDC first, then re-invokes.

### 2. Forecast submission reminder

Surface the Railway forecasting app URL: `https://tribeforecasting-production.up.railway.app` with a one-line "submit before EOD Sunday" reminder in the Slack DM. No automation; no skill exists for submission. The reminder is the deliverable.

### 3. Invoke `tribe-account-eow-update` per active engagement

Sequentially invoke the sub-skill once per active engagement code. Wait for each invocation to complete before starting the next so the Slack DM preview links accumulate cleanly in Harper's DM rather than racing.

For each invocation:
- Pass the engagement code as input
- Capture the Slack DM message link the sub-skill returns
- Record ✅ with link, or ❌ with the sub-skill's surfaced failure reason

Do not inline the sub-skill's logic (MEDDPICC delta synthesis, RAG colors, the visibility rules in `tribe-shared` projects, the Source Quotes Scratchpad). Those belong to `tribe-account-eow-update`; the orchestrator only invokes and records status.

**Pattern B graceful degradation.** GEV and Syneos have no Project OS memory directory. The sub-skill will run against engagement README + vault notes only. Expect a ⏳ status with a note like "Pattern B: no Project OS memory; recap drawn from vault README + this week's daily briefs."

### 4. Invoke `gm-eow-email`

After all per-engagement EOW posts have generated, invoke `gm-eow-email`. It writes the Tom email draft to `40-Briefs/weekly/[DRAFT] {today}-eow-tom.md` with its own drafting-notes footer.

Record ✅ with the file path, or ❌ with the sub-skill's surfaced failure reason.

### 5. Send the Friday wrap Slack DM to Harper

Compose the checklist DM using `.claude/skills/gm-cadence-prep/templates/friday-wrap-slack-dm.md`. Send to Harper's Slack DM (user_id `U0AR9GQAU06`) via the slack send-message capability (look up via `tool_search "slack send"`). Send direct, not draft.

The DM must include:
- ✅ / ⏳ / ❌ status per step (SFDC hygiene, forecast reminder, each engagement EOW post, Tom email)
- File paths and Slack DM links for every artifact ready for review
- Explicit list of owed actions (SFDC fixes if any, review-and-post per-engagement drafts, review-and-send Tom email, submit forecast Sunday)
- Sub-skill failures called out specifically: name the engagement code or skill, name the failure reason surfaced by the sub-skill, name the manual next step

## Conventions (Friday mode)

- **Delegation pattern.** Friday mode is an orchestrator. Do not re-derive what `gm-eow-email` or `tribe-account-eow-update` already encode. If the format of a per-engagement EOW post or the Tom email needs to change, that change belongs in the wrapped skill, not in this orchestrator.
- **Sequence matters.** SFDC hygiene first (so downstream skills see current data). Per-engagement EOW posts next (Harper reviews them while the Tom email is generating). Tom email last. Then the orchestrator's own checklist DM.
- **Failure reporting is specific.** If a sub-skill errors, the checklist DM names the sub-skill, the engagement code (if applicable), and the failure reason: not "something went wrong." Harper finishes the failed step manually.
- **No auto-retry.** If a sub-skill fails, mark ❌ and move on. Harper decides whether to re-invoke or finish by hand.
- **Active engagements only.** Iterate `10-Engagements/{code}/README.md` where `status: active`. Today that's `ge-vernova`, `syneos-health`, `kkr-insurance`. Paused/shadow accounts get a one-line "skipped: {reason}" in the Sources block: no EOW post attempt.
- **No new vault file.** Friday mode produces a Slack DM. The vault artifacts that land are the sub-skills' own outputs at their canonical paths.

## Failure modes (Friday mode)

- **Sub-skill missing.** If `gm-eow-email` or `tribe-account-eow-update` is not discoverable, stop before invoking either. Tell Harper which is missing and ask whether to proceed with the available one or abort.
- **Sub-skill errors mid-run.** Mark that step ❌ in the checklist DM with the verbatim failure reason. Continue to the next step (a failed per-engagement post does not block the Tom email).
- **Engagement folder missing or malformed.** Skip that engagement's EOW post, mark ❌ with "engagement folder not readable at `10-Engagements/{code}/`", surface in the DM's Sub-skill failures section.
- **No active engagements.** Skip step 3 entirely. The Tom email still runs (it has its own internal handling for empty active book). DM checklist shows "no active engagements: per-engagement EOW posts skipped."
- **SFDC hygiene blocker (any ❌).** Do not run steps 3-4. Send a minimal DM with the hygiene flags and the explicit instruction to fix SFDC then re-invoke `/cadence-prep friday`.
- **Slack send fails for the final checklist DM.** Write the DM body to `40-Briefs/weekly/{today}-friday-wrap-undelivered.md` and surface the path in the conversation so Harper can paste it manually.
- **Invoked outside Friday.** Confirm before running: "wrap for the ISO week ending today?" Per-engagement EOW posts and the Tom email both anchor to "this week's" range.

## Outputs (Friday mode)

- **Primary:** Slack DM to Harper (user_id `U0AR9GQAU06`) - the wrap checklist.
- **Sub-skill artifacts** (at their canonical paths, not produced by this mode):
  - `tribe-account-eow-update` × N active engagements → N Slack DM previews in `D0ARBJLCTM0`
  - `gm-eow-email` → `40-Briefs/weekly/[DRAFT] {today}-eow-tom.md`
- **Side effects:** none. Friday mode is read-only against engagement READMEs and SFDC; all writes happen inside the wrapped skills.

## Verification

### Sunday mode

For Sunday 2026-05-03 (first vault-file run):
1. Run `/cadence-prep sunday`.
2. Confirm output lands at `40-Briefs/weekly/2026-W19-sunday-prep.md`.
3. Spot-check: priority stack matches Brandon's 5/1 framing (GEV #1, Elliott #2, KKR #3). KKR 5/7 call appears prominently. Mike (Syneos) CIO call appears as Harper's first solo customer call. C&W shows as shadow-only (no commitments listed for Harper).
4. Read time ≤ 2 minutes. If longer, the week is genuinely complex; if dramatically longer, the skill is over-synthesizing.
5. Re-validate against the same week's Monday daily brief once it's generated: the Sunday prep priorities should anchor Monday's brief's "Suggested priorities" until/unless live signal overrides.

For the Slack DM addition (first run Sunday 2026-05-17):

6. Confirm a DM lands in Harper's Slack from the skill run.
7. DM contains: day-shape sentence with Monday meetings, one section per active engagement (GEV, Syneos, KKR; no HP, no C&W section), each with exactly top-3 numbered items, plus a workstreams section with up to 3 items across `45-Initiatives/*`.
8. Every top-3 item cites its source (`(README §X)`, `(Project OS action #N)`, etc.); no source-less items.
9. Engagement order reflects target-day time-urgency (most urgent first), not priority-stack order.
10. Newly-added high-stakes meetings (≤72hr old) carry ⚠️. Meaningful absences are called out.
11. Sources footer is tighter than the vault file but honest about gaps.
12. Format reference: the 5/10/2026 → 5/11 DM at `https://tribeai.slack.com/archives/D0ARBJLCTM0/p1778453498676589` is the canonical baseline. Future runs should match its shape (day-shape line → color-tagged engagement top-3 → workstreams → sources footer) even as content changes.

### Monday mode

For Monday 2026-05-11 (first run):

1. Run `/cadence-prep monday` early Monday morning, before the 9:30 AM ET deal-review prep block.
2. Confirm vault output lands at `40-Briefs/weekly/2026-W20-monday-gtm.md`.
3. Vault file contains: top-of-mind, announcements/shout-outs (or `_None._`), pipeline snapshot table, quarter forecast with explicit gap-to-goal, 2-3 key deals + support asks, "watching this week", sources footer.
4. Slack DM lands. First block is Harper's weekly priorities (personal note; inline source citations OK). Second block is GTM Weekly talking points (top-of-mind + key deals + support). Third block is the workstreams "watching" cut.
5. Forecast-week flag is true (the Tue 5/12 Bi-Weekly Pipeline Review is on the calendar).
6. `Goal = $3M Q2`, `Best Case = additional on top of Commit`. No double-counting.

### Tuesday mode

For Tuesday 2026-05-12 (first run):

1. Run `/cadence-prep tuesday` Monday evening or Tuesday morning.
2. Confirm vault output lands at `40-Briefs/weekly/2026-W20-tuesday-forecast.md`.
3. `bi_weekly_status: on` in frontmatter (the Tue 5/12 4pm event exists).
4. Vault file contains: pipeline hygiene table flagging kill/downgrade/re-stage candidates, MEDDPICC walk per active engagement with current vs prior color and delta arrow, coverage self-score, EB-access red flags, Harper's #1 commitment.
5. Slack DM lands evening-before or morning-of. Per active engagement: tally + delta + top weakness + coverage self-score. EB red flags surfaced separately. #1 commitment named with deliverable + deadline + source.
6. Vault MEDDPICC files are unchanged after the run (this skill is read-only against scoring).
7. KKR section shows the "vault scoring lags Project OS" note if Project OS has movement post-snapshot.

### Friday mode

For Friday 2026-05-15 (first run):

1. Run `/cadence-prep friday` around 3:00 PM ET.
2. SFDC hygiene check completes first. Any ❌ blocks downstream skills until Harper fixes.
3. `tribe-account-eow-update` invoked once per active engagement (ge-vernova, syneos-health, kkr-insurance). Each produces a Slack DM preview link.
4. `gm-eow-email` invoked after all per-engagement posts. Tom email draft lands at `40-Briefs/weekly/[DRAFT] 2026-05-15-eow-tom.md`.
5. Checklist Slack DM lands in Harper's DM with ✅/⏳/❌ per step, all artifact paths/links, owed actions, and the Railway forecast app reminder.
6. Sub-skill failures (if any) name the skill, engagement code, and surface the failure reason; no generic "something went wrong".
7. No new vault file from Friday mode itself: only sub-skill artifacts at their canonical paths.
8. **Skill-to-skill invocation is untested**: validate on the first real Friday run that Skill-invoked-from-inside-Skill composes cleanly. If it doesn't, the SKILL.md instructions degrade gracefully into "tell Harper to run `/eow` and `/tribe-account-eow-update {code}` per engagement manually" with the same checklist DM tracking what's owed.
