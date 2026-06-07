---
name: gm-eow-email
description: Generate Harper Foley's Friday end-of-week (EOW) email to Tom Lee in Brandon's standard Tribe AI format, sharpened to Harper's discipline. Pulls this week's calendar, Sybill calls, Slack priority-channel activity, prior daily briefs, every active engagement README and CLAUDE.md, the KKR Project OS memory directory if present, and recent vault updates, then synthesizes into the canonical EOW template (Pipeline with S2/S4 progressed + critical-only ≤2-line notes per deal / Q{N} Forecast / This Week as status-news + Next Week's Goals / Support Needed / optional Team Shoutouts) at `40-Briefs/weekly/[DRAFT] YYYY-MM-DD-eow-tom.md`. Use when Harper says "EOW email", "weekly email to Tom", "Friday email", "wrap the week for Tom", "end-of-week update for Tom", or invokes `/eow`. Do not use for engagement-specific Slack EOW posts (use `tribe-account-eow-update`), Sunday week-ahead prep (use `gm-cadence-prep sunday`), or daily briefs.
---

# GM End-of-Week Email

Harper's Friday cadence email to Tom Lee (direct manager). Tom relays the operating picture up to Brandon (Head of GTM), so the email's job is to give Tom a clean, accurate read on Harper's accounts in Brandon's standard format — Pipeline movement, Q2 forecast, last-week vs. next-week, asks, shoutouts.

A good EOW email is **scannable in 90 seconds, written in 15 minutes**. Brandon's training video says the upper bound is 30 minutes — past that, the email is doing too much. When the synthesis spirals, cut narrative paragraphs and trust the structure.

**The bar throughout is what Tom needs to stay apprised of, not a record of everything Harper did.** Report the news: the key actions that *landed* and the key next steps, not the activity log. Every line earns its place. If a section reads as a list of tasks, it's too long — cut to the status-changing facts.

## When to use

- Friday afternoon, before Harper closes for the week
- Phrases that should trigger: "EOW email", "weekly email to Tom", "Friday email", "wrap the week for Tom", "end-of-week update for Tom", "/eow"
- Re-runs allowed if a late-Friday meeting materially changes the picture (overwrite the draft)

## When NOT to use

- Engagement-specific Slack EOW post → `tribe-account-eow-update` (different audience, different format)
- Sunday week-ahead prep → `gm-cadence-prep sunday`
- Daily brief → `gm-daily-brief`
- Mid-week status to a single stakeholder → ad-hoc, not this skill
- "What did we discuss with {client}?" → direct Sybill/Granola lookup

## Inputs

- **Today's date** — default local today, America/New_York. If invoked outside Friday, ask Harper to confirm: "draft for the ISO week ending today?"
- **Target ISO week** — derived from today (e.g., 2026-05-08 → `2026-W19`, range Mon 5/4 → Fri 5/8).
- **Existing draft** — if `40-Briefs/weekly/[DRAFT] {YYYY-MM-DD}-eow-tom.md` already exists, ask Harper: overwrite, append a re-run section, or write to `[DRAFT] {YYYY-MM-DD}-eow-tom-rerun-HHMM.md`.
- **Q2 (current quarter) goal** — read from `~/.claude/projects/-Users-harperfoley-Library-Mobile-Documents-iCloud-md-obsidian-Documents-Tribe-AI/memory/user_role.md` (currently $3M for Q2 2026; verify the line is still current). Quotas rotate quarterly — re-prompt if the memory line looks stale (>1 quarter old).
- **CQ / NQ weighted pipeline numbers** — pull from SFDC if accessible; if not, ask Harper to paste the hover values from the Pipeline and Bookings Forecast dashboard (`https://tribeai.lightning.force.com/lightning/r/Dashboard/01ZHu000002PSJ2MAO/view`). These are weighted, not raw deal-sum — don't infer from deal sizes.

## Steps

Numbered. Each step says what data to pull, what tool category to use (look up specific tools via `tool_search`), and what to write where. If a step fails, record for the drafting-notes footer — do not silently skip.

### 1. Establish target week + check for existing draft

- Compute ISO week from today. Range is Mon → Fri of that week.
- Check whether `40-Briefs/weekly/[DRAFT] {today}-eow-tom.md` exists. If yes, ask Harper: overwrite / re-run-suffix / append.
- Read the prior week's EOW draft if present (`40-Briefs/weekly/[DRAFT] {last-friday}-eow-tom.md`). Its **Next Week's Goals** section is reference context for what Harper was aiming at; this week's **This Week** section names the key events that actually landed (not a goal-by-goal status walk).
- Pull the prior week's **Q{N} forecast bands** (Closed / Commit / Best Case / Closing Next Week) from the prior EOW. These are the carry-forward baseline — see step 8.

### 2. Read this week's daily briefs

- Glob `40-Briefs/daily/*.md` for files in the Mon–Fri range.
- For each: parse "Suggested priorities", "Open threads I owe", "Engagement pulse", and any "Override candidate" notes.
- Compute what landed vs. slipped vs. shifted. This is the source for the **This Week** list — the material, status-changing events (the news), not every action taken — and informs which of last week's **Next Week's Goals** were achieved (used for Harper's framing context, not as a goal-by-goal status walk in the email).

### 3. Pipeline state — SFDC + engagement READMEs

- For each active engagement (read `10-Engagements/CLAUDE.md` for the canonical list):
  - Read `10-Engagements/{code}/README.md` for current state + last material event + commercial shape.
  - Read `10-Engagements/{code}/CLAUDE.md` for client-specific watch-outs (informs Pipeline Notes voice — e.g., KKR external-facing language constraints from the reframe).
  - Read `10-Engagements/{code}/meddpicc.md` if present — the most recent score gives stage signal.
  - **Pattern A (today: KKR only):** also read `~/Projects/{code}/{code}-project-os/memory/project-state.md` and the most recent `memory/daily/*.md` for fresher state than the vault.
- Identify what advanced to **S2 or S4** specifically this week (Brandon's training: those are the only stages that go in "Pipeline Progressed"). S3 movements, new logos at S1, and stage holds do NOT go in this section — they go in Pipeline Notes. When the SFDC stage between S1 and S2 is genuinely ambiguous (e.g., the deal has the qualifying signals for S2 but the SFDC record may still show S1), use the "S1/S2 → {deal name}" format so Harper can resolve in the Deal Review block.
- For **every deal in the active pipeline**, compose a Pipeline Note. **Coverage is the rule: every live pipeline deal Tom is tracking gets a note — a new qualifying account silently omitted is a gap.** The bar per note is what Tom needs to stay apprised of on that deal, not a log of everything Harper did.
  - **Active deals:** at most **two short lines**, one atomic fact per line, each prefixed with a descriptive scannable label so Tom's eye can jump (`State:`, `Contracting:`, `Key insight:` — choose the labels that fit the deal).
    - **State:** where the deal is + the next gate/milestone, in one line.
    - **One optional second line:** the single most critical other fact — contracting status, the one risk, or the one derived insight. Pick one; do not stack three.
  - **Keep-warm / qualifying / out-of-quarter deals:** collapse to a one-line state + an `Out of Q{N}` line. Mark brand-new accounts `(new)` after the name.
- **Cut, don't include:** long `Confirmed:` enumerations (keep the single fact that matters), dated meeting play-by-play, MEDDPICC tally or per-dimension state (lives in Salesforce), timeline references like "5/19 PM call landed X," and **any ask that's already in Support Needed** — don't repeat a Tom-ask as a "Help needed" pipeline line; it lives in Support Needed only. The Slack per-account EOWs carry the detail; the Tom email reads decision-shaped, not narrative.

### 4. This week's meetings + Sybill + Granola

- Pull the week's calendar (Mon → today).
- For each material client or internal-strategic meeting (skip recurring team Q&As, training, personal blocks):
  - Pull the Sybill conversation (or Granola note) if it exists.
  - Capture: who attended, what was decided, what changed in the deal shape.
- Cross-reference with the daily briefs' "Engagement pulse" — daily briefs already synthesized most of this; the Sybill/Granola pull is for events the daily brief missed (especially Friday meetings post-brief).

### 4a. Source Quotes Scratchpad (pre-draft attribution gate)

Before any line of the email body gets composed, build an internal scratchpad of verbatim source quotes for every claim that names a person, attributes a quote, asserts call ownership, or references a product. **Do not skip this step under time pressure — this is where Brandon-vs-Branden and Cloud-vs-Claude-Coach errors get caught.**

For each candidate claim, capture:

- **Source** (Sybill transcript ID / Granola note ID / Slack channel + ts / Gmail thread ID / vault file path)
- **Speaker** (verbatim, as the source spells the name — do not normalize from memory)
- **Timestamp** (HH:MM or message ts)
- **Verbatim quote** (the exact words the source contains — never paraphrase at this step)

Use the scratchpad to verify name spellings against the canonical source before any proper noun lands in the draft. Specifically:

- Tribe-internal names: cross-check against Slack `slack_search_users` if any doubt (Brandon Bright, Tom Lee, Trevor Douglass, Hadley Riegel, Victor Miller, Noah Gale, Jaclyn Rice Nelson).
- Tribe products: only **SkillForge, SkillHub, Claude Coach** are Tribe products (per root `CLAUDE.md` § Product Source-of-Truth). Any other product name in a draft is a customer product and must trace to the engagement's vault or Project OS.
- Client-side stakeholder names: verify against the engagement README and the most recent Sybill/Granola transcript that mentions them. Do not write a client name from memory — pull it.

**If a claim cannot be sourced to a verbatim quote, do one of three things:** drop it, soften it to non-attributed framing ("the team discussed X" instead of "Brandon said X"), or flag it in the Drafting notes footer as `⚠️ Unsourced: {claim}` for Harper's review.

The scratchpad lives in the drafting process only — do not persist it into the final email body. The footer's `**Name verification**` line below is the visible audit trail.

### 5. Slack priority channels — Mon → today

- Reference canonical priority channel list in `.claude/references/data-surfaces.md`.
- Pull the week's activity for each.
- Capture: deal-affecting threads, stakeholder-engagement signals (silence too — e.g., a sponsor who hasn't replied is a forecast risk worth flagging in Forecast Notes), team shoutout candidates.

### 6. Gmail VIP scan — Mon → today

- Pull threads in the week's range from VIP senders (Brandon, Tom, Trevor, Hadley, Victor, plus engagement-domain senders).
- Surface only threads that change the deal picture or create new asks.
- Auth-failure handling: surface `⚠️ Gmail auth expired` in drafting notes; never silently ship without Gmail data.

### 7. Recent vault updates — Mon → today

- `find` for engagement README/CLAUDE.md modifications in the week's range. Read the diffs (or the current state if no diff is available) — these reflect Harper's own deal-shape revisions and often carry context the daily briefs missed.
- Check `40-Briefs/weekly/` for any mid-week prep files (Sunday prep especially) — those frame what Harper *intended* the week to be.

### 8. Compute Q{N} forecast math

- **Closed-Won this quarter**: sum of S5 deals closed since quarter start. If SFDC isn't accessible, ask Harper.
- **Goal**: quarterly quota from memory (`user_role.md`).
- **Commit**: deals Harper is committing to close this quarter. From SFDC `Forecast Type = Commit` filter or Harper's read.
- **Best Case**: deals plausible to close this quarter beyond Commit. **Convention: Best Case is the *additional* amount on top of Commit, not inclusive.** Total ceiling = Commit + Best Case (per Brandon's training and Notion example: `$215k, $385k = $600k`).
- **Closing Next Week**: deals with a real next-week close date in SFDC. Honest "None" is acceptable — never inflate.
- **Carry-forward rule:** If no deal actually closed or moved into Commit / out of Commit this week, the **Closed / Commit / Best Case bands stay aligned to the prior EOW**. Only the underlying *breakdown* (which deals make up Best Case) updates. Don't shift the bands on vibes alone — Tom uses week-over-week consistency to read trajectory.
- **Notes — gap-to-goal math is required.** Format the Notes section as four bullets:
  1. `Commit holds at ${Commit}M.` (with optional one-line honest hedge in Harper's voice, e.g., "Uncertain if Syneos will be able to sign by Q{N} close.")
  2. `Best Case = {deal A breakdown} + {deal B breakdown}.`
  3. `Gap to goal = ${gap}M.`
  4. `Realistic Q{N} close depends on (a) {condition A} AND (b) {condition B}.`
  Brandon's training is explicit: this is where the analytical work lives. Specific deals + conditions, not vibes.

### 9. Identify support needs + team shoutouts

- **Support Needed** — explicit, deal-specific asks. Read recent daily briefs' "Open threads waiting on others" + recent meeting decisions for owed follow-ups. One to three bullets max — too many dilutes the signal.
  - **No "Tom:" prefix on bullets.** The email is to Tom; prefixing his name is redundant. Default-to-Tom items have no prefix. If a Support Needed bullet is for someone else (Brandon, Trevor, Noah), name them explicitly: `Brandon: ...` or `Noah: ...`.
  - **No GEV-input asks routed to Tom.** GEV is being worked between Harper, Pete, and Rosaria; Tom doesn't track GEV state. GEV pipeline notes stay in the email (Tom should know the state), but input requests for GEV go to Pete or Rosaria DMs separately, not the EOW Support Needed section. This may change if Tom takes GEV back on; until then, no GEV asks here.
- **Team Shoutouts is optional.** Drop the section entirely when nothing heroic happened this week; don't manufacture wins. If included: name + one-line specific contribution per bullet, sourced from this week's Sybill calls, daily brief "Engagement pulse" entries, and Slack channel activity. Three bullets ideal when the section is justified.

### 10. Compose the email

- Path: `40-Briefs/weekly/[DRAFT] {today}-eow-tom.md` (note the bracketed `[DRAFT]` prefix per `feedback_file_naming_status_prefix`).
- Use the template at `.claude/skills/gm-eow-email/templates/eow-email.md`.
- Frontmatter: `recipient: Tom Lee <tom@tribe.ai>`, `week: {ISO}`, `date_range: {Mon} to {Fri}`, `status: DRAFT`, `draft_generated: {today}`.
- Subject line: `Harper EOW — Week of {Mon date}` (e.g., `Harper EOW — Week of 5/4`).
- Salutation: `Tom,`
- Sign-off: `Harper` (no title, no signature block — Brandon's example doesn't have one).

### 11. Drafting notes footer

After the email body, write a `## Drafting notes (delete before sending)` section covering:
- **Numbers needing verification** — anything inferred (stages, weighted pipeline, Q2 goal if memory was stale).
- **Stale or risky lines** — places where the prior week's framing might no longer hold (e.g., a stage advancement that needs SFDC confirmation before send).
- **Cuts considered but not made** — be transparent about what almost got included so Harper can re-add if he disagrees.
- **Sources status** — ✅ / ⚠️ / ❌ per source pulled, one-line each.
- **Name verification** — ✅ / ⚠️ per proper noun and product reference in the draft, with the verbatim source quote each was checked against (from the Step 4a scratchpad). Any ⚠️ items must explain what couldn't be confirmed.

This footer is for Harper's review only and gets stripped before he sends to Tom.

## Outputs

- **Primary**: `40-Briefs/weekly/[DRAFT] YYYY-MM-DD-eow-tom.md` — draft email + drafting notes footer.
- **Side effects**: none. Skill is read-only against engagement READMEs, daily briefs, and Project OS memory. Don't update those files; that's the job of the engagement-specific skills and `gm-evening-debrief`.

## Conventions

- **Recipient is Tom Lee, not Brandon.** Per `feedback_weekly_email_recipient`. Brandon's *format*, Tom's *inbox*. The calendar reminder may say "Send Brandon" — that title is stale; don't be misled.
- **Format is Brandon's standard, sharpened to Harper's discipline.** Per `reference_weekly_email_format` for the canonical Brandon shape, then the sharpening rules below. Sections in order: Pipeline (Progressed S2/S4, Weighted Pipeline, Notes — critical-only, ≤2 lines per deal) → Q{N} Forecast (Closed/Goal, Commit/Best Case, Closing Next Week, Notes) → This Week / Next Week (key events from this week + forward-looking goals) → Support Needed → optional Team Shoutouts. Match the Notion example's terse style (`https://www.notion.so/tribeai/AEs-Weekly-Email-How-to-Prepare-1a04f38daa2080a4b226f5bed9ffa7bd`).

### Sharpening rules (Harper's discipline on top of Brandon's standard)

- **Pipeline Notes: critical-only, ≤2 lines per deal, scannable.** Lead each line with a descriptive label (`State:` plus at most one of `Contracting:` / `Key insight:` / the one risk). Every live pipeline deal gets a note — a new qualifying account omitted is a gap. Keep-warm / qualifying deals collapse to a state line + `Out of Q{N}`. Cut long `Confirmed:` enumerations, dated play-by-play, and any ask already in Support Needed (don't duplicate it as a "Help needed" line). The bar: what Tom needs to stay apprised of, not everything Harper did.
- **No MEDDPICC details in Pipeline Notes.** MEDDPICC scoring lives in Salesforce; the Tom email never duplicates tally, deltas, or per-dimension state.
- **S2/S4 only in Pipeline Progressed.** Brandon's training video is explicit. S3 movements, new logos, stage holds, and back-pedals all belong in Pipeline Notes. Use "S1/S2 → {deal}" when SFDC stage between S1 and S2 is genuinely ambiguous.
- **Best Case math: additional on top of Commit.** Total ceiling = Commit + Best Case. Notion example: `$215k, $385k = $600k`.
- **Q{N} forecast carry-forward.** When no deal actually moved this week, the Closed / Commit / Best Case bands stay aligned to the prior EOW; only the deal breakdown changes. Tom reads week-over-week consistency for trajectory.
- **Forecast Notes carry gap-to-goal math** in four bullets: Commit hold (with optional honest hedge) → Best Case breakdown → Gap to goal → "Realistic Q{N} close depends on (a) ... AND (b) ...". Specific deals + conditions, not vibes.
- **This Week = the news, not the activity log.** One tight bullet per account that *materially moved* this week — the events that changed the account's status (a stage gate proposed, a contract returned, sign-off cleared, a discovery call booked). Enumerate what *landed*, not everything Harper did. **Only accounts with material status movement get a bullet** — keep-warm pings, routine qualifying touches, and accounts with no status change are not news and don't get a line (they're already in Pipeline Notes). Three to four bullets is typical; if every active account has a bullet, it's too long. **Section labels: "This Week" + "Next Week's Goals" (combined header "This Week / Next Week"),** not "Last Week's Goals vs Actuals." No DONE/IN PROGRESS/NOT DONE status tags — just the events.
- **Team Shoutouts is optional.** Drop the section entirely when nothing heroic happened. Don't manufacture wins; nothing routine counts.
- **No "Tom:" prefix on Support Needed bullets.** Email is to Tom. Default-to-Tom items have no prefix. Items for someone else are explicitly named (`Brandon: ...`, `Noah: ...`).
- **No GEV-input asks to Tom.** GEV is worked between Harper, Pete, and Rosaria; Tom isn't in the GEV thread. GEV pipeline notes stay (Tom should know state), but input requests on GEV go to Pete or Rosaria DMs, not the EOW.

### Account-specific reminders

- **KKR language constraints.** Per `project_kkr_reframe`: external-facing copy uses "commercial liability sourcing"; never say "ALM lost" or quote walked-back numbers (100bps, billions). Tom is internal but he relays to Brandon — be precise.
- **Syneos workshop is no-cost.** Per `project_syneos_coverage_handoff`: workshop is Tribe-funded (3- or 4-day in-person). The 3-month engagement is the first paid step. $5M+ full engagement is Phase 2 positioning — don't lead with it. Workshop proposal must be reviewed by Elliott Management before going to Syneos.

### General

- **Voice**: terse, operator-grade, no narrative paragraphs. The Notion example is ~200 words total; Harper's emails run longer when pipeline activity warrants but stay terse per section. If a section runs long, cut to one-line bullets.
- **HP / C&W**: drop entirely from Pipeline Notes unless there's material movement. Both are off the active book (paused / shadow). HP may surface as a Support Needed bullet if a follow-up is owed.
- **Honesty about uncertainty**: "Closing Next Week: None" is honest and acceptable. Honest hedges in the Notes section are welcome ("Uncertain if Syneos will be able to sign by Q2 close"). Inflated commit numbers erode trust faster than a small honest commit.
- **Drafting notes are review-only.** Always include them; Harper deletes before send.

## Failure modes

- **SFDC inaccessible**: prompt Harper to paste CQ/NQ weighted hover values from the dashboard. Don't infer weighted from deal sums — weighted ≠ raw.
- **Q2 quota memory stale**: if `user_role.md` quota line was set in a prior quarter, prompt Harper for the current Q2 number rather than carrying forward a wrong target.
- **Daily briefs missing for some days**: still draft — note the gap in drafting notes (`⚠️ No daily brief for Wed 5/6 — last-week actuals derived from Sybill + vault only`).
- **Pattern A engagement Project OS memory empty**: fall back to the engagement vault README; flag the source gap in drafting notes.
- **Late-Friday meeting post-draft**: re-run is allowed; overwrite with a `-rerun-HHMM` suffix or append a `## Update — HH:MM ET` section.
- **Stage inference uncertainty**: if you can't verify a stage from MEDDPICC + recent activity, note it in drafting notes with explicit "verify in SFDC before send" rather than asserting.
- **Attribution can't be sourced**: if a name, quote, product reference, or call-ownership claim has no verbatim source in the Step 4a scratchpad, drop the line or surface `⚠️ Unsourced: {claim}` in drafting notes. Never synthesize a name or quote that no source supports — this is the root cause of the Brandon/Branden and Cloud/Claude Coach errors the skill is built to prevent.

## Verification

Same dogfood pattern as `gm-daily-brief`: validate over **3 consecutive Fridays** before considering the skill stable. Each Friday Harper:

1. Invokes the skill.
2. Reads — does the draft take ≤ 90 seconds to scan? Are the Pipeline numbers correct? Is gap-to-goal math present and defensible?
3. Spot-checks the drafting notes — are the flagged uncertainties the *right* uncertainties?
4. Tracks edit-distance from draft → sent — high edit-distance signals the skill missed the picture.

Tune the steps, the template, and the conventions between runs.
