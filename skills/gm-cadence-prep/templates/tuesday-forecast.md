---
mode: tuesday-forecast
week: {YYYY-Www}
meeting_date: {YYYY-MM-DD}
bi_weekly_status: {on | off}
generated_at: {HH:MM ET}
sources: [calendar, sybill-deals, vault-meddpicc, vault-snapshots, project-os, prior-tuesday-prep, prior-sunday-prep]
---

# Week {YYYY-Www} — Tuesday Forecast Prep

{Meeting date: Tue {YYYY-MM-DD} 4:00-5:00 PM ET. Bi-weekly status: {on | off}.}

{If bi-weekly is OFF this week, lead with: *"This week is OFF the bi-weekly cadence. No Tuesday meeting. Generating this prep anyway for forecast hygiene; Harper to confirm before relying on it."*}

## The meeting's shape

{One paragraph. What's the read going into this Tuesday: which deal is the live story, what changed since last bi-weekly, which field is most at risk of pushback from Tom / Brandon, what Harper needs to commit to in the Momentum & Accountability block.}

{If any source is ❌ or ⚠️, lead with a one-line heads-up here. Especially: missing snapshot for delta computation, missing canonical meddpicc.md, stale SFDC stage.}

## Pipeline hygiene — per active engagement

{Per Tom's agenda block #1 (10 min). Per-engagement table. Flag anything stale: SFDC close date in the past, no activity 14+ days, stage doesn't match recent calls.}

| Engagement | SFDC stage | Close date | Amount | Last touch | Flag |
|---|---|---|---|---|---|
| GE Vernova | {stage} | {date} | {$} | {date / event} | {✅ clean / ⚠️ stale / 🟥 needs re-stage} |
| Syneos Health | {stage} | {date} | {$} | {date / event} | {flag} |
| KKR Insurance | {stage} | {date} | {$} | {date / event} | {flag} |

**Kill / downgrade / re-stage candidates:**
- {engagement} — {recommendation} — {why} — {what to say in the meeting}
- {or `_None. All three deals defensible at current stage._`}

**"Why will this close in the timeframe we've committed to?" — per deal:**
- **GE Vernova** ({stage}, close {date}): {one-sentence defense}
- **Syneos Health** ({stage}, close {date}): {one-sentence defense}
- **KKR Insurance** ({stage}, close {date}): {one-sentence defense}

## MEDDPICC walk — full portfolio

{Per Tom's agenda block #2 (40 min, ~20 min per GM). Three sub-sections, one per active engagement. READ ONLY from canonical `meddpicc.md` + snapshots. Do not modify or generate scoring.}

### GE Vernova — {tally line, e.g. 2 🟩 / 5 🟨 / 1 🟥}

**Coverage self-score:** {strong / partial / gap} — {one-sentence rationale}

**Field-by-field** (current state, from `10-Engagements/ge-vernova/meddpicc.md`):

| Field | Now | Prior ({snapshot date}) | Delta | Defense line for the meeting |
|---|---|---|---|---|
| M — Metrics | {🟩/🟨/🟥} | {🟩/🟨/🟥} | {→ / ↑ / ↓} | {one-line speaking script} |
| E — Economic Buyer | {} | {} | {} | {} |
| D — Decision Criteria | {} | {} | {} | {} |
| D — Decision Process | {} | {} | {} | {} |
| P — Paper Process | {} | {} | {} | {} |
| I — Identify Pain | {} | {} | {} | {} |
| C — Champion | {} | {} | {} | {} |
| C — Competition | {} | {} | {} | {} |

**What moved since last snapshot ({prior date}):**
- {field} {prior color → current color} — {one-line why, with source citation}
- {or `_No movement. Last refresh {date}._`}

**EB-access red flag:** {yes / no} — {if yes: who is the EB, why Harper doesn't have direct access, what the path to access is}

**Likely pushback in the room:**
- {anticipated objection from Tom / Brandon} — {prepared response}

### Syneos Health — {tally line}

**Coverage self-score:** {strong / partial / gap} — {rationale}

**Field-by-field** (from `10-Engagements/syneos-health/meddpicc.md`):

| Field | Now | Prior ({snapshot date}) | Delta | Defense line |
|---|---|---|---|---|
| M — Metrics | {} | {} | {} | {} |
| E — Economic Buyer | {} | {} | {} | {} |
| D — Decision Criteria | {} | {} | {} | {} |
| D — Decision Process | {} | {} | {} | {} |
| P — Paper Process | {} | {} | {} | {} |
| I — Identify Pain | {} | {} | {} | {} |
| C — Champion | {} | {} | {} | {} |
| C — Competition | {} | {} | {} | {} |

**What moved since last snapshot ({prior date}):**
- {delta or `_None._`}

**EB-access red flag:** {yes / no} — {detail}

**Likely pushback:**
- {anticipated objection} — {response}

### KKR Insurance — {tally line}

**Coverage self-score:** {strong / partial / gap} — {rationale}

**Field-by-field** (from `10-Engagements/kkr-insurance/meddpicc.md`):

| Field | Now | Prior ({snapshot date}) | Delta | Defense line |
|---|---|---|---|---|
| M — Metrics | {} | {} | {} | {} |
| E — Economic Buyer | {} | {} | {} | {} |
| D — Decision Criteria | {} | {} | {} | {} |
| D — Decision Process | {} | {} | {} | {} |
| P — Paper Process | {} | {} | {} | {} |
| I — Identify Pain | {} | {} | {} | {} |
| C — Champion | {} | {} | {} | {} |
| C — Competition | {} | {} | {} | {} |

**What moved since last snapshot ({prior date}):**
- {delta or `_None._`}

**Pattern A note:** {if Project OS `memory/project-state.md` has movement that hasn't propagated to vault `meddpicc.md` yet, flag it here — e.g. "Functional-leader call landed 5/12 per Project OS daily log; vault scoring still pre-call. Surface in the meeting that scoring refresh is owed."}

**EB-access red flag:** {yes / no} — {detail; Neir = operational sponsor, William two-no-show pattern, funding gate KKR-Insurance vs. KKR-parent unconfirmed → this is Harper's biggest EB exposure}

**Likely pushback:**
- {anticipated objection} — {response}

## EB-access red flags — portfolio view

{Per Tom's pre-work ask: flag any deal where Harper doesn't have direct access to the Economic Buyer. Pull from each engagement's CLAUDE.md "People" / "Political reality" section + MEDDPICC EB field.}

- **{engagement}** — EB = {name + role} — Harper's access: {direct / via {intermediary} / none} — gap: {one line} — plan: {what Harper proposes in the meeting}
- {or `_No EB-access gaps. All three deals have direct or near-direct EB access._`}

## Momentum & Accountability — Harper's #1 commitment

{Per Tom's agenda block #3 (10 min). One named priority action for the next 2 weeks, with specific deliverable + deadline. Pull from open commitments + Sunday-prep priority stack.}

**#1 priority action ({next 2 weeks}):** {what Harper will name in the meeting}

- **Deliverable:** {specific artifact or event}
- **Deadline:** {date — must be before the next bi-weekly}
- **Why this one:** {one sentence — what's at stake, what it converts}
- **Source:** {engagement README §X / Project OS action #N / Sunday-prep priority #N}

**Runner-up commitments** (in case Tom presses for a second):
- {item} — {deliverable + deadline}
- {item} — {deliverable + deadline}

## What was committed last bi-weekly — and what landed

{If prior Tuesday prep exists, surface what Harper named as his #1 last time and whether it shipped. If first run, write `_First Tuesday-prep run. No prior commitments to track._`}

- **Last bi-weekly's #1:** {what Harper named}
- **Status:** {shipped / partial / slipped — with one-line evidence}
- **If slipped:** {what to say in the room — own it, don't dodge}

## Sources

- {✅ Calendar (Tuesday {date}, {N} events)}
- {✅ Sybill `list_deals` ({N} deals across Harper's book — GEV, Syneos, KKR)}
- {✅ Vault canonical MEDDPICC ({N}/3 engagements scored — `meddpicc.md` per engagement)}
- {✅ MEDDPICC snapshots ({N}/3 engagements with prior snapshot for delta — most recent + ~2 weeks prior)}
- {✅ Engagement READMEs + CLAUDE.mds ({N}/3 active)}
- {Pattern A — ✅ KKR Project OS `memory/project-state.md` + `memory/action-items.md` if path exists, ❌ otherwise}
- {✅ Prior Sunday-prep ({week file})}
- {✅ Prior Tuesday-prep ({week file or `none — first run`})}

{Use ⚠️ for partial / stale, ❌ for failed pulls. Always specify failure mode: "snapshot >2 weeks old", "auth expired", "MCP timeout", "path not found", "canonical meddpicc.md missing — no field-by-field walk possible".}

## Format rules

- **Read-only against MEDDPICC.** This skill never modifies `meddpicc.md` or snapshot files. If scoring is stale, surface that in the Sources footer and recommend Harper run `meddpicc-scorecard-update` separately.
- **Conservative defaults match the canonical file.** Don't soften scoring in the speaking script. If Metrics is RED in `meddpicc.md`, the defense line acknowledges RED and names what closes it.
- **Delta computation:** "prior" = the most recent snapshot in `reference/meddpicc-snapshots/` that pre-dates the last bi-weekly Tuesday. If no qualifying snapshot exists, write `_no prior snapshot — first scoring_` and skip the delta column.
- **Active engagements only.** GEV, Syneos, KKR. Skip HP (paused) and C&W (shadow). If a paused/shadow account has SFDC movement, surface it in the Pipeline Hygiene table only.
- **No em dashes.** Hyphens or rewrites only.
- **Voice:** operator-grade, terse, direct. Defense lines are speaking scripts — say them out loud to test cadence.
