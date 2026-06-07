---
name: gm-calendar-event
description: Create a Google Calendar event in Harper Foley's preferred GM format (bracketed `[TAG]` title plus a six-section HTML description covering GOAL / INPUTS / OUTPUT / TASK DETAILS / AGENDA / NOTES). Use whenever Harper says "create a calendar event", "add to my calendar", "add to calendar", "schedule a block", "schedule X on my calendar", "block time for X", "put this on the calendar", "make a calendar event", "calendar this", "put X at Y on my calendar", or any equivalent ask to create or schedule a calendar event. Do not use for reading or summarizing existing calendar events, for engagement READMEs / daily-brief writes, or for declining or updating events Harper does not own.
allowed-tools: mcp__claude_ai_Google_Calendar__create_event, AskUserQuestion, Read, Glob, Grep, Bash
---

# GM Calendar Event

Creates a Google Calendar event in Harper's standard GM format. Title is `[TAG] short descriptor`. Description is a six-section HTML block that lets Harper walk into the block with goal, inputs, output artifact, task details, agenda, and watch-outs already loaded.

The skill's job is to make the calendar event itself function as the prep doc. When Harper opens the event 30 seconds before, he should see what to produce, where the inputs are, what the agenda is, and the watch-outs, without bouncing to the vault.

## When to use

- Harper asks Claude to create, add, schedule, or block a calendar event
- Phrases that should trigger: "create a calendar event", "add to my calendar", "add to calendar", "schedule a block", "schedule X on my calendar", "block time for X", "put this on the calendar", "make a calendar event", "calendar this", "put X at Y on my calendar"
- Harper has just laid out a daily plan in conversation and asks to "put these on the calendar". Create one event per block, pulling content directly from the conversation.

## When NOT to use

- Reading or summarizing existing events: use calendar list/get tools directly
- Updating an event Harper does not own: suggest the parallel-prep-event workaround (see Edge cases)
- Writing engagement READMEs, daily briefs, or vault notes: those have their own skills
- Declining an event: use the calendar respond tool directly

## Inputs

What the skill needs to start. If any of these are missing from the conversation, batch them into one `AskUserQuestion` call rather than asking one at a time:

- **Start + end time** (or duration). Default timezone `America/Los_Angeles` unless Harper specifies.
- **TAG**: derive automatically when possible (see TAG derivation below). Ask only when ambiguous.
- **GOAL**: one or two sentences. The decision or output the block produces.
- **INPUTS**: file links, Sybill URLs, Slack archive URLs, external articles.
- **OUTPUT**: specific artifact path + description, or "No written artifact" for prep / reset blocks.
- **TASK DETAILS**: concrete imperative moves.
- **AGENDA mode**: solo block (no agenda) vs. live meeting (numbered agenda) vs. solo block with async DMs (mixed).
- **NOTES**: watch-outs, disambiguations, memory citations, tone guidance.

If Harper has already laid out a daily plan in context (a common case), pull every field from the conversation. Do not re-ask.

## TAG derivation

Map the engagement directory to the bracket tag. Match by checking which engagement code under `10-Engagements/{code}/` is referenced in conversation or inputs:

| Engagement directory | TAG |
|---|---|
| `ge-vernova` | `[GEV]` |
| `kkr-insurance` | `[KKR]` |
| `syneos-health` | `[Syneos]` |
| `hp` | `[HP]` |
| `cushman-wakefield` | `[C&W]` |

For non-engagement workstreams, use the workstream tag:

| Workstream | TAG |
|---|---|
| GM operating system / systematization | `[GM-OS]` |
| GTM weekly, pipeline, forecasting | `[GTM]` |
| Wrap blocks (end-of-day, end-of-week, reset) | `[Wrap]` |
| Outbound, prospecting | `[Outbound]` |
| Team management, 1:1 prep | `[Team]` |
| 1:1 meeting | `[1:1]` |
| Personal blocks (kids, family, errands) | `[Personal]` |

If a block doesn't fit any of the above, ask Harper for the TAG rather than guessing.

## Steps

### 1. Gather or auto-derive inputs

- Scan the recent conversation for: start/end times, engagement names, file references, Sybill URLs, Slack URLs, agenda hints, watch-outs.
- Auto-derive TAG from any engagement code match in the conversation.
- If two or more fields are still missing after the scan, ask via a single batched `AskUserQuestion`.

### 2. Construct the HTML description

Use exactly six sections, in this order, with these emoji and headers. Empty sections still render. Write `_None._` or a one-line equivalent rather than dropping the section.

```html
<p>🎯 <strong>GOAL</strong><br>
One or two sentences: what decision or output this block produces.</p>

<p>📥 <strong>INPUTS</strong></p>
<ul>
<li>Bulleted links to vault files, Sybill conversations, Slack archives, external articles.</li>
</ul>

<p>📤 <strong>OUTPUT</strong></p>
<p>Specific artifact (file path + brief description), or "No written artifact" for prep / reset blocks.</p>

<p>📋 <strong>TASK DETAILS</strong></p>
<ul>
<li>Concrete moves. Imperative voice.</li>
</ul>

<p>🤝 <strong>AGENDA</strong><br>
Live meeting: numbered or bulleted agenda. Solo block: "Solo block. No agenda." or "Solo block; async DMs to X." if there are async actions.</p>

<p>🗒️ <strong>NOTES</strong><br>
Watch-outs, disambiguations, memory citations as <code>memory_slug</code>, tone guidance.</p>
```

If OUTPUT has multiple artifacts, swap the `<p>` wrapper for a `<ul>` with one `<li>` per artifact.

### 3. Apply link conventions

- **Vault files**: `file:///Users/harperfoley/Library/Mobile%20Documents/iCloud~md~obsidian/Documents/Tribe%20AI/{path}`. Always URL-encode spaces as `%20`.
- **Sybill conversations**: `https://app.sybill.ai/conversations/{uuid}`.
- **Slack archive**: `https://tribeai.slack.com/archives/{channel-id}/p{ts}`.
- **Granola**: deep-link format is unconfirmed (per `reference_granola_deeplink`). Leave Granola UUIDs bare or use the title only.
- **Memory citations**: inline `<code>memory_slug</code>` matching the file name (minus `.md`) under `~/.claude/projects/-Users-harperfoley-Library-Mobile-Documents-iCloud-md-obsidian-Documents-Tribe-AI/memory/`.

### 4. Escape HTML-unsafe characters

- `<` and `>` inside content become `&lt;` and `&gt;` (e.g. `Tribe&lt;&gt;GEV`).
- `&` inside content becomes `&amp;` (e.g. `C&amp;W` if it appears inside body text; the TAG `[C&W]` lives in the title field, which is not HTML and stays literal).
- Quotes inside attribute values become `&quot;`. Curly quotes in prose render fine.

### 5. Sweep before calling the API

This is the gate. Do not call `create_event` until all six pass.

- [ ] No em dashes (Unicode U+2014). Replace with hyphen, semicolon, period, or rewrite the sentence.
- [ ] No en dashes (Unicode U+2013). Same replacement rule.
- [ ] No forbidden words from Harper's global preferences. The short list to scan for: `opt`, `dive`, `unlock`, `unleash`, `intricate`, `transformative`, `proactive`, `scalable`, `leverage`, `synergy`, `robust`, `facilitate`, `utilize`, `holistic`, `optimize`, `empower`, `disrupt`, `innovative`, `game-changing`, `cutting-edge`, `actionable`, `granular`, `mission-critical`, `ensure`, `crucial`, `vital`. If one appears, rewrite the line.
- [ ] All six sections present, in order, with the right emoji.
- [ ] All `<` and `>` inside body content escaped.
- [ ] Title follows `[TAG] short descriptor` pattern (unless this is an external meeting Harper does not own, in which case stop and see Edge cases).

A quick mechanical check: run `grep -nP '[\x{2013}\x{2014}]'` against the description string before sending. Any hits must be rewritten before the API call.

### 6. Call `create_event`

Look up the calendar create tool via `tool_search "calendar create"` if not in context. Required parameters:

- `summary`: the `[TAG] short descriptor` title.
- `startTime`, `endTime`: RFC3339 timestamps.
- `timeZone`: `America/Los_Angeles` unless Harper specified otherwise.
- `description`: the HTML block from Step 2, post-sweep.

For live meetings with attendees:

- `attendeeEmails`: list of invitee emails.
- `notificationLevel`: default is `EXTERNAL_ONLY` which is fine for most internal-plus-client meetings. Use `ALL` only when Harper explicitly wants everyone notified.

For recurring events (rare): `recurrenceData` accepts RRULE strings. Only set if Harper asks.

### 7. Report back

Return the event's `htmlLink` so Harper can verify in calendar. One line:

`Created: [TAG] short descriptor, Mon 5/12 2:00 to 3:00 PT, {htmlLink}`

If `create_event` fails, return the error verbatim plus the proposed event payload so Harper can fix and retry. Do not silently re-attempt with mutated inputs.

## Checklist before calling create_event

Run this top to bottom every time. The skill's failure mode is shipping an event with an em dash or a forbidden word in the description.

1. Title matches `[TAG] short descriptor`.
2. Six sections present, in order: GOAL, INPUTS, OUTPUT, TASK DETAILS, AGENDA, NOTES.
3. Em-dash sweep clean (U+2014).
4. En-dash sweep clean (U+2013).
5. Forbidden-word sweep clean.
6. All `<` / `>` inside body content escaped.
7. Vault links URL-encoded (`%20` for spaces).
8. Memory citations wrapped in `<code>...</code>`.
9. Timezone is `America/Los_Angeles` unless overridden.
10. For meetings: attendees populated, notification level set deliberately.

## Canonical examples

### Example A: Solo deep-work block (GM-OS, 2026-05-12)

**Title:** `[GM-OS] Phase 1 scaffold + Brandon slip-and-recover`

**Description (HTML):**

```html
<p>🎯 <strong>GOAL</strong><br>
Draft v0 of the Notion account-page scaffold (fields + sub-pages + databases) and close the loop with Brandon on the slipped EOW 5/8 commitment with substance, not apology.</p>

<p>📥 <strong>INPUTS</strong></p>
<ul>
<li><a href="file:///Users/harperfoley/Library/Mobile%20Documents/iCloud~md~obsidian/Documents/Tribe%20AI/45-Initiatives/gm-operating-system/README.md">GM-OS initiative README: Charter §6 + Phase 1 open list</a></li>
<li><a href="https://app.sybill.ai/conversations/55424fcb-8679-482e-a3bd-75c59efe7c4b">Sybill: 5/11 GTM Weekly</a>. Brandon's "as soon as we write it down it's old".</li>
</ul>

<p>📤 <strong>OUTPUT</strong></p>
<ul>
<li><code>45-Initiatives/gm-operating-system/notes/2026-05-12-notion-scaffold-v0.md</code>: field-level draft of the account-page scaffold</li>
<li>Slack DM to Brandon Bright with the scaffold link + a new shipped-by date</li>
</ul>

<p>📋 <strong>TASK DETAILS</strong></p>
<ul>
<li>Brain-dump current GM new-account moves from memory (Charter §6 step 1, before reading top-rep examples)</li>
<li>Draft scaffold fields: owner, stage, sponsor, MEDDPICC scorecard, Sybill links, top-of-page key-point summary</li>
</ul>

<p>🤝 <strong>AGENDA</strong><br>
Brandon DM only. No live conversation. Tone: own the slip in one line, show what's shipped, name the new date.</p>

<p>🗒️ <strong>NOTES</strong><br>
This is the absorbed deal-process-sop scope (per GM-OS README "absorbs" frontmatter). Per <code>feedback_systematization_scope</code>, systematize the plumbing (context-assembly), not the strategy.</p>
```

### Example B: Pre-meeting reset block (GEV, 2026-05-12)

**Title:** `[GEV] Reset + 3pm walk-in re-read`

**Description (HTML):**

```html
<p>🎯 <strong>GOAL</strong><br>
Walk into the 3pm GEV with the stakeholder map, current MEDDPICC tally, and the last call's open questions cleanly loaded. No new artifact; this is a mental reset.</p>

<p>📥 <strong>INPUTS</strong></p>
<ul>
<li><a href="file:///Users/harperfoley/Library/Mobile%20Documents/iCloud~md~obsidian/Documents/Tribe%20AI/10-Engagements/ge-vernova/README.md">GEV engagement README: stakeholder map §4 + current state</a></li>
<li><a href="file:///Users/harperfoley/Library/Mobile%20Documents/iCloud~md~obsidian/Documents/Tribe%20AI/10-Engagements/ge-vernova/meddpicc.md">GEV MEDDPICC scorecard</a></li>
</ul>

<p>📤 <strong>OUTPUT</strong></p>
<p>Aligned mental model walking into the 3pm. No written artifact.</p>

<p>📋 <strong>TASK DETAILS</strong></p>
<ul>
<li>Re-read README §4 stakeholder map. Andy Rector canonical, not Rechter.</li>
<li>Skim MEDDPICC tally; note any dimension yellow-to-green candidates the call might unlock.</li>
<li>Pull the last Sybill call's open questions list.</li>
</ul>

<p>🤝 <strong>AGENDA</strong><br>
Solo block. No agenda.</p>

<p>🗒️ <strong>NOTES</strong><br>
Disambiguate: Branden Smith is GEV-side, Brandon Bright is Tribe-side (per <code>reference_branden_smith_gev</code>). Andy Rector spelling confirmed (per <code>reference_andy_rector_gev</code>).</p>
```

### Example C: Workstream block with external articles (GTM, 2026-05-12)

**Title:** `[GTM] AI ROI framework: Infrastructure stub + reviewer chase`

**Description (HTML):**

```html
<p>🎯 <strong>GOAL</strong><br>
Stub the AI ROI framework's "infrastructure cost" section and send chase DMs to the three reviewers who haven't replied to the v0 draft.</p>

<p>📥 <strong>INPUTS</strong></p>
<ul>
<li><a href="file:///Users/harperfoley/Library/Mobile%20Documents/iCloud~md~obsidian/Documents/Tribe%20AI/45-Initiatives/ai-roi-framework/README.md">AI ROI framework initiative README</a></li>
<li><a href="file:///Users/harperfoley/Library/Mobile%20Documents/iCloud~md~obsidian/Documents/Tribe%20AI/45-Initiatives/ai-roi-framework/notes/2026-05-08-v0-draft.md">v0 draft (sent to reviewers 5/8)</a></li>
<li><a href="https://a16z.com/ai-infrastructure-cost-curve/">a16z: AI infra cost curve (Mar 2026)</a></li>
</ul>

<p>📤 <strong>OUTPUT</strong></p>
<ul>
<li><code>45-Initiatives/ai-roi-framework/notes/2026-05-12-infrastructure-stub.md</code>: section draft, 200 to 400 words</li>
<li>Three Slack DMs to outstanding reviewers</li>
</ul>

<p>📋 <strong>TASK DETAILS</strong></p>
<ul>
<li>Skim the a16z piece for the cost-per-token trajectory chart; cite the source inline.</li>
<li>Write the stub at vendor-agnostic abstraction. No SkillForge / SkillHub / Claude Coach references here; this is framework-level.</li>
<li>DM chase: tone is light, single-question ("any blockers on review by Fri?"), not a re-pitch.</li>
</ul>

<p>🤝 <strong>AGENDA</strong><br>
Solo block; async DMs to three reviewers.</p>

<p>🗒️ <strong>NOTES</strong><br>
Per <code>reference_tribe_products</code>, Tribe products are SkillForge / SkillHub / Claude Coach. Keep this framework abstract: don't anchor to Tribe products in the framework body.</p>
```

## Edge cases

- **Recurring event**: `recurrenceData` accepts RRULE strings. Only set if Harper explicitly asks (e.g. "every Tuesday for 6 weeks"). Default is single occurrence.
- **External meeting where Harper is not the organizer**: do NOT create a duplicate event. Instead, propose creating a parallel "prep" event 15 to 30 minutes before, titled `[TAG] Prep: {external meeting title}`, with the standard six-section description. Confirm with Harper before creating the prep block.
- **Personal block** (kids, family, errands): use `[Personal]` tag. Description still uses the six sections but they collapse: GOAL is one-liner, INPUTS often `_None._`, OUTPUT often `_None._`, TASK DETAILS lighter, AGENDA is `Personal block. No agenda.`, NOTES often `_None._` or a single watch-out (e.g., commute time).
- **Time zone other than PT**: if Harper says "ET" or "CT" or names a city, use the matching IANA zone (`America/New_York`, `America/Chicago`, etc.). Default stays `America/Los_Angeles`.
- **Conflicting event already on calendar**: surface the conflict in the report-back line but still create the event (Harper can resolve). Do not block on conflict detection.
- **Title pattern unclear**: if no engagement code matches and no workstream tag fits, ask Harper for the TAG. Do not invent one.

## Outputs

- **Primary**: a Google Calendar event in Harper's calendar with the title pattern and six-section HTML description.
- **Return value**: one-line confirmation including the event's `htmlLink`.
- **Side effects**: none to the vault. The skill is calendar-write-only; it does not update engagement READMEs, daily briefs, or memory files.

## Conventions

- **Title is plain text, not HTML.** `[TAG]` is literal brackets in the summary; do not HTML-escape there.
- **Description is HTML.** Google Calendar renders it. Plain-text descriptions render but lose structure.
- **Six sections, every time, in order.** Skip nothing. Empty sections collapse to a single line, not removal.
- **Memory citations are documentation, not decoration.** Cite only when the citation changes the block's behavior (a disambiguation, a tone rule, a watch-out from prior feedback). Decorative citations dilute signal.
- **Voice**: calm operator. Plain language. Imperative in TASK DETAILS. No forbidden words, no em or en dashes, no hedging.
- **Default timezone**: `America/Los_Angeles`. Override only when Harper says so or names a non-PT city.

## Failure modes

- **`create_event` returns an auth error**: surface the error verbatim; do not silently retry. Tell Harper to re-auth Google Calendar.
- **Em dash sneaks through the sweep**: Step 5 is the last gate. If an em dash lands in a created event, that's a skill bug. Fix the event via `update_event` and update the skill's sweep step.
- **TAG ambiguity**: better to ask once than create with the wrong tag. The tag is the search handle Harper uses to find blocks in calendar week-view.
- **External meeting accidentally created as a duplicate**: the "External meeting" entry under Edge cases exists to prevent this. If it happens, delete the duplicate and create the parallel prep event instead.
