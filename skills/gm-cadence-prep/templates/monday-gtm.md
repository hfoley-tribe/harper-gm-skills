---
mode: monday-gtm
week: {YYYY-Www}
range: {YYYY-MM-DD to YYYY-MM-DD}
generated_at: {HH:MM ET}
forecast_week: {true | false}
sources: [calendar, slack, sybill-list-deals, sybill-calls, engagement-readmes, project-os, prior-sunday-prep, prior-daily-briefs]
---

# Week {YYYY-Ww} - Monday GTM Prep

{Range: {Mon date} → {Sun date}. GTM Weekly: Mon 2:00–3:00 PM ET, Brandon organizing.}

## The shape of the meeting

{One paragraph. What Harper plans to say, what he needs from the room, anything different from last week. Note if Tom is OOO this week or if any non-standard agenda items are floating. Flag explicitly if this is a forecast-submission week.}

{If any source is ❌ or ⚠️, lead with a one-line heads-up here.}

---

## Top of mind (round-the-room, ~1 min)

{1-2 sentences Harper says when Brandon opens the meeting. Pull from this week's commitments + any signal from engagement READMEs that landed since Sunday prep. Mention newly-added high-stakes meetings or shifts in the last 24-48 hours.}

> {Verbatim draft Harper can speak.}

---

## Announcements / shout-outs (pre-submit to Brandon via Slack)

{0-2 items. Skip the section if nothing landed worth surfacing. Each item is one sentence Brandon can read aloud or paraphrase. Include the source signal so Harper can verify before submitting.}

- **{Announcement / shout-out}** - {source: Sybill call, Slack thread, engagement README §X}.

{Or:}

_Nothing to submit this week._

---

## Pipeline snapshot - owned book

{Per-account row. Pull from Sybill `list_deals` + engagement README frontmatter (`meddpicc_tally`, `last_reviewed`, `status`, `stage`). Compute week-over-week delta vs the prior Sunday-prep file and last week's daily briefs.}

| Account | Stage | Amount | Close | MEDDPICC | Δ vs last week |
|---|---|---|---|---|---|
| GE Vernova | {S?} | ${amount} | {date} | {tally} | {delta} |
| Syneos Health | {S?} | ${amount} | {date} | {tally} | {delta} |
| KKR Insurance | {S?} | ${amount} | {date} | {tally} | {delta} |
| HP (paused) | {S?} | ${amount} | {date} | {tally} | {delta or "no movement"} |

{Notes below the table for any row where the delta needs context. Keep to 1 sentence each.}

- **GE Vernova:** {what moved, why}.
- **Syneos:** {what moved, why}.
- **KKR:** {what moved, why}.

---

## Quarter forecast

{Pull from Sybill `list_deals` for the owned book + engagement README frontmatter. Goal = $3M Q2 per Brandon's standard. Best Case = additional on top of Commit per memory `reference_weekly_email_format`.}

- **Commit:** ${amount}
- **Best Case (additional on top of Commit):** ${amount}
- **Closed-Won (QTD):** ${amount}
- **Goal:** ${amount}
- **Gap to goal:** ${commit + closed-won vs goal math, one line}

{Forecast-week flag.}

> {Forecast submission status this week - confirmed forecast-submission week per Tue {date} Bi-Weekly Team Pipeline Review on calendar. Harper to submit before GTM Weekly per the all-day calendar reminder.}

{Or, if not a forecast week:}

> _Not a forecast-submission week. Next submission: {date}._

---

## Key deals + support needed (2-3 to surface)

{Deals Harper plans to raise during the Sales Team Update block (~20 min). Each: one-liner state, one-liner support ask. Pull from engagement READMEs + Pattern A Project OS state.}

### {Deal 1 - engagement name}
- **State:** {one sentence}.
- **Ask:** {what Harper needs from Brandon / Tom / the room}.

### {Deal 2 - engagement name}
- **State:** {one sentence}.
- **Ask:** {what Harper needs}.

### {Deal 3 - engagement name (optional)}
- **State:** {one sentence}.
- **Ask:** {what Harper needs}.

---

## What I'm watching this week

{1-3 bullets, highest-signal events. Pull from Sunday-prep priority stack if present.}

- {Event / signal} - {what to watch for}.
- {Event / signal} - {what to watch for}.
- {Event / signal} - {what to watch for}.

---

## Day shape - Monday

{Chronological. Lead with the 8:30 AM personal weekly-priorities block (self-direction, not team-facing). End with the 2pm GTM Weekly.}

- **8:30 AM ET** - Set weekly priorities (personal note, not a team Slack post; no SOP today says these go to a team channel).
- **9:30 AM ET** - Prep for weekly deal review (self-block).
- **{HH:MM ET}** - {meeting}.
- **2:00–3:00 PM ET** - GTM Weekly (Brandon).

---

## Sources

- ✅ Calendar (Monday + this week, {N} events)
- ✅ Sybill `list_deals` ({N} deals across owned book)
- ✅ Engagement READMEs ({N} read; frontmatter parsed for MEDDPICC + stage)
- {Pattern A entry - ✅ if KKR Project OS `memory/project-state.md` + `memory/action-items.md` read, ❌ if path missing}
- {Prior Sunday-prep - ✅ if `40-Briefs/weekly/{this-week}-sunday-prep.md` exists, ❌ otherwise}
- ✅ Last week's daily briefs ({N} briefs read for week-over-week delta)
- ✅ Slack priority channels (last 24-48hr for shifts since Sunday prep)
- ✅ Sybill calls (last 7d, {N} calls - for any signal that didn't make Sunday prep)

{Use ⚠️ for partial / stale, ❌ for failed pulls. Include specific failure mode and remediation.}
