---
name: tribe-account-eow-update
description: Generate Harper's end-of-week Slack update for a Tribe AI client account. Reads Project OS memory artifacts (action items, daily logs, decisions, project-state, MEDDPICC scorecard if present), produces the canonical EOW format, and posts to Harper's DM as a preview before broader sharing. Use whenever Harper asks for an "EOW", "EOW update", "end-of-week update", "weekly recap", "Friday update", "wrap the week on [account]", or any equivalent for a client account — even if she doesn't explicitly say "Slack message". Also use when she's wrapping up a Friday session and the conversation has arrived at a recap-shaped output that she'd plausibly want to push out.
---

# Tribe Account EOW Update

Harper sends end-of-week (EOW) Slack updates for each active account she runs as GM at Tribe AI. The format is consistent across accounts — codifying it here so it's reproducible without re-deriving structure each Friday.

## What this skill does

1. Reads the project's memory artifacts to ground the update in current reality
2. Formats the output in Harper's canonical EOW structure (defined below)
3. Respects visibility rules from the project's `CLAUDE.md` — no financials in `tribe-shared` outputs
4. Posts to Harper's DM (`D0ARBJLCTM0`) as a preview using `slack_send_message`. She reviews and re-posts to the account channel manually after.

## Inputs to read

These live in Project OS repos at predictable paths. Read what's present; gracefully skip what isn't. Don't synthesize content for missing artifacts — say what's missing in the conversation back to Harper instead.

| Artifact | Path | Used for |
|---|---|---|
| CLAUDE.md | `CLAUDE.md` | Visibility flag, account name, team, current priorities, active risks |
| Action items | `memory/action-items.md` | Open items per owner; what's blocking; recent resolutions |
| Daily logs | `memory/daily/*.md` | What happened this week — meetings, decisions, sent emails |
| Decisions log | `memory/decisions.md` | Decisions made this week worth surfacing |
| Project state | `memory/project-state.md` | Risk + stakeholder posture (most recent snapshot) |
| MEDDPICC scorecard | `docs/project/private/meddpicc-scorecard.md` | Harper-maintained convention; only present if she's tracking the deal in MEDDPICC. Skip the MEDDPICC section if absent. |

The scorecard path is Harper-specific — not part of base Project OS scaffolding. Don't synthesize a MEDDPICC view from raw memory if no scorecard exists; that's noise.

## Output structure

Slack standard markdown (`**bold**`, `_italic_`, `-` bullets). Send to D0ARBJLCTM0 via `slack_send_message`.

```
**EOW [Account Name] + Next Steps**

**This week**
- [3-5 bullets of concrete events with dates: meetings held, decisions made, emails sent, syncs run]

**Where we stand**
- [Open gating items / who's responding to what / waiting-on-others on client side]
- [Items in flight on Tribe side — pinged legal, drafting X, etc.]

**MEDDPICC**
- **M** (Metrics) [prior]🟢/🟡/🟥 → [now]🟢/🟡/🟥 — [one-line state]
- **E** (Economic Buyer) [prior] → [now] — [state]
- **D** (Decision Criteria) [prior] → [now] — [state]
- **D** (Decision Process) [prior] → [now] — [state]
- **P** (Paper Process) [prior] → [now] — [state]
- **I** (Identified Pain) [prior] → [now] — [state]
- **C** (Champion) [prior] → [now] — [state]
- **C** (Competition) [prior] → [now] — [state]

**Risk + posture I want input on**
Honest read: [name the pressing risk in 1-2 sentences].
[Evidence — reschedule patterns, no-shows, silence, etc. — that grounds the read].
[Posture proposal, framed conditionally: "We may need to keep warm rather than push hard. We might need to DQ for now and revisit on a monthly ping cadence if [specific trigger condition by named date]." Close with empathy for client state where appropriate: "they might need to first figure out what they actually want before we can fully help them."] @[Person] @[Person] — would value your read before I commit either way.

**Ask for [Person]**
@[Person] — [framing of what Harper wants from them, e.g., "discovery questions I'm thinking we need answered next call to either qualify, advance, or DQ. What am I missing?"]

[Topic] — [one-line question]
[Topic] — [one-line question]
[...]

**Next steps**
- Monitor [specific signal] through EOD [named day]
- If [condition]: [action]
- If not: [action with named day-of-week trigger]
- [Optional: stakeholder-input ask, e.g., "Get team's input on discovery questions for next call"]
```

### Format details — read these carefully

**MEDDPICC order is fixed and non-negotiable:** M, E, D-Criteria, D-Process, P, I, C-Champion, C-Competition. This is Harper's preferred canonical order — matches her MEDDPICC representation memory.

**RAG colors:** Use these emojis specifically — `🟢` (green) definitively answered · `🟡` (yellow) partial / line of sight to green · `🟥` (red) unanswered. Do not substitute text equivalents or other colors.

**MEDDPICC delta format:** Each dimension shows prior-state arrow current-state, then a one-line summary. If a dimension didn't move, still show it (`🟡 → 🟡 — [state]`) so the full posture is visible. **Do not include an aggregate tally header** (e.g., "0🟢/5🟡/3🟥 → 2🟢/4🟡/2🟥") — the per-dimension list speaks for itself, and the aggregate is redundant in a Slack distillation. The scorecard file maintains an aggregate tally, but the EOW Slack message does not.

**Tone — Harper's voice:**
- Direct, terse, no filler
- Hedges where uncertain: "Honest read:", "feels like", "may need to", "might need to"
- Em-dashes for compound thoughts and reframes
- No italics for emphasis — plain text reads more direct in Slack
- No corporate-speak, no buzzwords, no "exciting" / "great"

**De-absolutize claims that aren't yet certain.** When the underlying memory says "X has gone to Y" but the actual evidence is partial or inferential, soften to "some portion of X has gone to Y." Don't drop in absolutist taglines ("Tribe is boxed out") unless the evidence supports them. Better to be a hedge too cautious than a hedge too confident — the EOW gets quoted in sales channels and the framing sticks.

**Conditional DQ framing.** Never propose a hard action ("move to DQ") without naming the trigger condition. The pattern Harper validated: "We might need to DQ for now and revisit on a monthly ping cadence **if we don't see [specific signal] by [named date]**." The conditional makes the proposal reviewable rather than declarative — teammates can disagree with the trigger without disagreeing with the principle.

**Empathetic close where appropriate.** When the deal is stalling because the client hasn't figured out what they want, acknowledge that explicitly. "They might need to first figure out what they actually want before we can fully help them" frames the situation in a way that's true to the client's perspective, not just ours. This isn't softening for its own sake — it correctly attributes the bottleneck.

**The Asks section is flexible.** Some weeks have no specific ask, some have asks of multiple people. Match what's actually needed; drop the section if there's nothing. When asking for input on discovery questions, frame as "What am I missing?" not "Please review."

**@-mentions are literal text in the draft, not real user IDs.** The Slack tool doesn't auto-resolve names. Send the preview as-is with literal `@Brandon @Tom @Victor`, then explicitly remind Harper after sending that she'll need to retype them as real @-mentions when she pastes to the team channel.

**No emojis in body content** other than the RAG colors (🟢🟡🟥). No 🚀🎯💡 etc. Harper's house style is plain text.

## Source Quotes Scratchpad (mandatory, build before drafting)

Before composing any line that names a person, attributes a quote, asserts call ownership, or references a product, build an internal scratchpad of verbatim source quotes from the project's memory artifacts. **This is a hard gate. The EOW gets quoted in sales channels and the framing sticks — attribution errors at this layer end up in `#sales` threads where they're hard to retract.**

For each candidate claim, capture:

- **Source** (specific memory artifact path: `memory/daily/2026-05-09.md`, `memory/action-items.md`, `memory/decisions.md`, etc., or the Sybill/Granola/Slack reference if available)
- **Speaker / actor** (verbatim, as the source spells the name — do not normalize from your own memory)
- **Timestamp or date** (the day or meeting the claim is sourced from)
- **Verbatim quote or paraphrase boundary** (the exact words the source contains; if paraphrasing, mark it explicitly so it doesn't leak as a quote)

Verification rules:

- **Tribe-internal names**: cross-check spellings (Brandon Bright, Tom Lee, Trevor Douglass, Hadley Riegel, Victor Miller, Noah Gale, Jaclyn Rice Nelson). Call transcripts mis-spell these; the project memory may already contain the error. When in doubt, the canonical spelling lives in Slack — `slack_search_users` is the tiebreaker.
- **Tribe products**: only **SkillForge, SkillHub, Claude Coach** are Tribe products (per root `CLAUDE.md` § Product Source-of-Truth). Any other product name in the draft is a customer product and must trace to this engagement's `CLAUDE.md` or memory artifacts.
- **Client-side stakeholder names**: verify against `CLAUDE.md` and the most recent daily log or meeting note that mentions them. Never write a client name from memory — pull it.
- **Call ownership**: action-items.md frequently attributes calls to specific people; cross-check against the daily logs before writing "X ran Y" or "X is owed Z."

**If a claim cannot be sourced to a verbatim quote or memory artifact**, do one of three things: drop it, soften to non-attributed framing ("the team discussed X" instead of "Brandon said X"), or include it as a flagged item in the Delivery footnote (Step 5) so Harper can verify before reposting.

The scratchpad lives in the drafting process only — do not include it in the Slack message body.

## Visibility check (mandatory, read before drafting)

Read `CLAUDE.md` for the `Visibility` flag. The rules:

- **`tribe-shared`:** No financials anywhere in the output. This means no contract values, no billing rates, no margins, no cost estimates, no payment terms, and no specific dollar figures Harper or the client has surfaced (including opportunity-size figures the client gave). Refer to opportunity sizing qualitatively — for example "Neir's #2 by readiness, possibly #1 by monetary value per his framing" rather than a dollar amount. The visibility rule applies even if memory artifacts contain figures.
- **`internal`:** Financials are permissible but use judgment. If unsure, leave them out.

This is a hard rule because Harper treats EOW outputs as artifacts that may be forwarded or pasted into shared channels. A figure that lands in a tribe-shared channel can't be unsent.

## Why each section exists (theory of mind)

Don't treat the structure as decoration — each section earns its place:

- **This week** — concrete grounding so teammates can verify the recap against their own memory
- **Where we stand** — surfaces what's gating progress so the team knows where to apply pressure
- **MEDDPICC** — quantifies trajectory deal-by-deal; the delta is what matters week-over-week, not the absolute state
- **Risk + posture** — forces a decision-shaped read every Friday rather than letting accounts drift on autopilot. This is the most important section. The posture should propose an action (push, hold, DQ, escalate) tagged to someone for input
- **Asks** — explicit work request; without this, the team doesn't know what's wanted of them
- **Next steps** — Harper's commitments + conditional logic so people know what triggers escalation

If a section feels empty, don't pad it — drop it (except MEDDPICC, which only drops if the scorecard is missing).

## Format validation

The 2026-05-08 KKR EOW — the first message produced in this exact format — was endorsed by Brandon Bright (Head of GTM) in a Tribe sales channel as a model end-of-week review. The structure encoded in this skill is the as-validated version, not a proposal. Treat the format as load-bearing; preserve the section ordering, the MEDDPICC per-dim arrow style, the conditional DQ framing, and the empathetic close pattern.

## Action-item extraction discipline

When reading `action-items.md` to summarize "what happened this week," do not translate "client surfaced X as a concern/criterion" into "Tribe drafts a deliverable on X for client." Client-side asks are not Tribe-side deliverables. The recap should reflect what was actually said, committed, or done — not inflated translations. (Harper has corrected this pattern multiple times; carrying it forward.)

## Delivery

Default flow:

1. Read all available inputs from the project
2. Draft the message in the exact structure above
3. Send to Harper's DM `D0ARBJLCTM0` via `slack_send_message` (standard markdown: `**bold**` for section headers; no italics)
4. Return the message link in the conversation
5. Flag any of the following that apply:
   - The @-mention literal-text caveat (always flag — user needs to retype before posting to team channel)
   - Missing inputs that limited the recap (e.g., "no daily log for 5/9 yet, week recap leans on 5/7-5/8")
   - Stale claims found in memory that you didn't trust (e.g., "project-state.md still references X but the daily log shows Y; used Y in the recap")
   - **Name verification** — list ✅ / ⚠️ per proper noun and product reference in the draft, with the verbatim source quote each was checked against (from the Source Quotes Scratchpad). Any ⚠️ items must explain what couldn't be confirmed. If a claim was dropped because it couldn't be sourced, note it here too (`⚠️ Dropped: {claim}`). Never silently synthesize a name, quote, or product reference that no memory artifact supports.

Don't send to the team channel directly. Harper does that step herself after reviewing the preview.

## When NOT to use this skill

- One-off retrospectives, status emails to clients, or QBR-style summaries — those are different artifacts with different audiences
- Mid-week check-ins or daily updates — this skill is for Friday EOW only
- Internal Tribe team updates that aren't account-specific — this skill is account-scoped

If Harper asks for something that's adjacent but not the standard weekly post, ask one clarifying question rather than forcing this format.
