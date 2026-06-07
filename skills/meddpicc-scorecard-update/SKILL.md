---
name: meddpicc-scorecard-update
description: Update Harper's existing MEDDPICC scorecard with new evidence from recent project memory artifacts (daily logs, decisions, call transcripts, action items). Identifies dimension state changes with quoted evidence, refreshes the tally + history, updates the Hadley 5-disqualifier filter, refreshes the adversarial review questions. Writes the canonical vault file at `[vault]/10-Engagements/{code}/meddpicc.md`, snapshots the prior version to `reference/meddpicc-snapshots/YYYY-MM-DD.md`, then propagates to (1) the Project OS synced copy at `~/Projects/{code}/{code}-project-os/docs/project/private/meddpicc-scorecard.md` for Pattern A engagements, and (2) the vault engagement README's `meddpicc_tally` + `meddpicc_updated` frontmatter so the dashboard at `00-System/HOME.md` stays in sync. Use whenever Harper asks to "update the scorecard", "refresh MEDDPICC", "roll the scorecard forward", "rescore [account]", "update MEDDPICC for [account]", "what's changed in MEDDPICC since [date]", or any equivalent. Also use when she's wrapping up a Friday session before EOW posting and the scorecard hasn't been refreshed yet — week-over-week deltas are what give the EOW its trajectory line.
---

# MEDDPICC Scorecard Update

Update Harper's existing MEDDPICC scorecard with new evidence from project memory. Pairs with `meddpicc-scorecard-bootstrap` (creates v0) and `tribe-account-eow-update` (reads the scorecard for the EOW MEDDPICC delta line).

## Canonical-vs-synced architecture

> **Read this before touching any path.**

MEDDPICC is **GM-layer scoring** and lives in the vault for every engagement, per `10-Engagements/CLAUDE.md`. There is one canonical file per engagement and (for Pattern A engagements) one synced copy:

| File | Role | Pattern A | Pattern B |
|---|---|---|---|
| `[vault]/10-Engagements/{code}/meddpicc.md` | **Canonical.** Single source of truth. Skill reads + writes here first. | ✅ | ✅ |
| `[vault]/10-Engagements/{code}/reference/meddpicc-snapshots/YYYY-MM-DD.md` | Frozen history. Skill creates one per refresh, before overwriting canonical. | ✅ | ✅ |
| `~/Projects/{code}/{code}-project-os/docs/project/private/meddpicc-scorecard.md` | **Synced copy** of the vault scorecard. Provides in-repo visibility when working from Project OS. Skill propagates here. | ✅ | n/a |
| `[vault]/10-Engagements/{code}/README.md` frontmatter (`meddpicc_tally`, `meddpicc_updated`) | **Denormalized** for the dashboard's Active Engagements table. Skill propagates here. | ✅ | ✅ |

**Rule:** Never hand-edit the synced Project OS copy. Edit the vault file (or run this skill) — the next run will overwrite the Project OS copy.

## Inputs

### Resolve `{code}` first

The engagement code drives every path. Resolve it in this order:

1. **From CWD if running inside a Project OS repo** (`~/Projects/{code}/{code}-project-os/`):
   ```bash
   CODE=$(basename "$PWD" | sed 's/-project-os$//')
   ```
2. **From CWD if running inside the vault engagement folder** (`[vault]/10-Engagements/{code}/`):
   ```bash
   CODE=$(basename "$PWD")
   ```
3. **From an explicit arg** if Harper said "refresh MEDDPICC for kkr-insurance" — use that string verbatim.
4. **Otherwise, ask** rather than guess. Don't run if `{code}` is ambiguous.

Sanity-check: `{code}` should match an existing folder under `[vault]/10-Engagements/`. If it doesn't, stop and surface.

### Required: canonical scorecard

Read `[vault]/10-Engagements/{code}/meddpicc.md`. Note the `last_updated` frontmatter date and the current aggregate (`aggregate_green / aggregate_yellow / aggregate_red`) — that's the baseline.

If the file doesn't exist, **do not proceed** — invoke `meddpicc-scorecard-bootstrap` instead.

### Read for new evidence (since `last_updated`)

For **Pattern A** engagements (vault README frontmatter `pattern: A`):

| Artifact | Path | Used for |
|---|---|---|
| Daily logs | `~/Projects/{code}/{code}-project-os/memory/daily/*.md` | New meetings, attendance, sent emails, decisions |
| Decisions log | `~/Projects/{code}/{code}-project-os/memory/decisions.md` | Anything ratified since last update |
| Call transcripts | `~/Projects/{code}/{code}-project-os/memory/calltranscripts/*.md` | Verbatim evidence with timestamps |
| Action items | `~/Projects/{code}/{code}-project-os/memory/action-items.md` | What got resolved, what's still open, status changes |
| Project state | `~/Projects/{code}/{code}-project-os/memory/project-state.md` | Current risk + stakeholder posture (cross-check against scorecard for drift) |
| GM-layer notes | `[vault]/10-Engagements/{code}/notes/*.md` | Harper's private GM thinking — political reads, watch-outs |

For **Pattern B** engagements (vault README frontmatter `pattern: B`):

| Artifact | Path | Used for |
|---|---|---|
| Meeting synthesis | `[vault]/10-Engagements/{code}/meetings/*.md` | Recent meetings, attendance, decisions |
| GM notes | `[vault]/10-Engagements/{code}/notes/*.md` | Private GM thinking, political reads |
| Deliverable drafts | `[vault]/10-Engagements/{code}/deliverables/*.md` | What's been sent, what's in flight |
| Live MCPs | Sybill, Granola, Slack | Pull on demand for evidence the vault may not have yet |

For **shadow** engagements (`pattern: B (shadow)`): this skill shouldn't run. Surface to Harper.

## What to update

Walk each MEDDPICC dimension in canonical order: **M, E, D-Criteria, D-Process, P, I, C-Champion, C-Competition**. For each:

1. **Read the existing field row** in the scorecard's field-by-field table. What's the current status, score/value, and evidence?
2. **Scan new memory for evidence** that would change the state — promote (🟥 → 🟨, 🟨 → 🟩), demote (🟩 → 🟨, 🟨 → 🟥), or refine the claim within the same color.
3. **If the dimension changed:** rewrite the row with the new status, refreshed score/value, refreshed evidence with citation, and refreshed "Converts to GREEN when" criterion if the goalpost moved.
4. **If the dimension didn't change:** leave the row alone. Do not rewrite for style.

Match the scorecard's existing structure. The current vault format (see `kkr-insurance/meddpicc.md`) uses a 5-column table: `Field | Status | Score / Value | Evidence | Converts to GREEN when`. If the engagement uses a different layout (older scorecards may use 5 columns of `Status | Dimension | Present state | Gap to close | Action`), follow the existing layout — don't unilaterally migrate format.

### Demotion is allowed and important

Dimensions can go backward. If a sponsor stops attending calls, Champion can demote 🟨 → 🟥. If a competitive vendor expands footprint, Competition can demote toward worse positioning. The scorecard is a state-of-reality snapshot, not a trophy case — be willing to mark regression when evidence supports it.

## Update the aggregate header

The vault format carries the aggregate in two places:

1. **YAML frontmatter** (top of file):
   ```yaml
   last_updated: YYYY-MM-DD
   aggregate_green: N
   aggregate_yellow: N
   aggregate_red: N
   prior_snapshot_date: YYYY-MM-DD     # the previous "last_updated"
   prior_aggregate: "NG / NY / NR"     # the previous aggregate string
   ```
2. **In-body header lines** below the H1:
   ```
   **Aggregate:** N 🟩 / N 🟨 / N 🟥
   **Last refreshed:** YYYY-MM-DD (Harper Foley) — one-line summary of the change driver
   **Trend vs. prior:** One-line summary of the delta vs. the prior aggregate, with the date.
   ```

Update both. If the engagement's scorecard uses an older "Tally as of [date]: ..." single-line format, preserve that format and just update the values.

## Update the Hadley 5-disqualifier filter (if present)

Some scorecards carry a Hadley 5-disqualifier filter section (Business Value, Scope, Altitude, Budget, Incentives). If the file has this section, walk each disqualifier and update its score (✅ / ⚠️ / 🟥) based on new evidence. Note movement explicitly in the Notes column — for example: *"Improved from 🟥 — Neir absorbed pod math without pushback on 5/7."*

Update the disqualifier count and "kill / not kill territory" verdict at the bottom of the table.

If the file doesn't have this section, don't add one — Harper drives whether/when to introduce it.

## Refresh adversarial review questions (if present)

Some scorecards carry three adversarial review questions to surface unresolved tensions. If present, review the existing three:
- Are any obsolete? (the situation that prompted them has resolved or moved on)
- Are there sharper tensions worth surfacing now?
- Is the skeptic's posture appropriately advancing?

Refresh up to all three. It's fine to keep one if it's still load-bearing — but mark it as carried forward (*"(carried from prior version — still load-bearing)"*).

If the file doesn't have this section, don't add one.

## Snapshot before overwrite

Per `10-Engagements/CLAUDE.md` refresh process, **before overwriting** the canonical vault file:

1. Copy current `[vault]/10-Engagements/{code}/meddpicc.md` to `[vault]/10-Engagements/{code}/reference/meddpicc-snapshots/YYYY-MM-DD.md` where `YYYY-MM-DD` is the **prior** `last_updated` date (the snapshot represents the state being frozen).
2. In the snapshot, strip the "Refresh triggers" section (it's a forward-looking artifact, not historical) and prepend a `> Frozen historical record. Do not edit.` blockquote.
3. If `reference/meddpicc-snapshots/` doesn't exist, create it.
4. If a snapshot for that date already exists (unusual — multiple refreshes same day), append a `-{n}` suffix to the second one.

## Write back to the canonical vault file

Use the `Edit` tool with targeted old_string → new_string for each changed row. **Do not rewrite the whole file.** Edits should look like a focused diff, legible to Harper and to any future review.

Update the YAML frontmatter dates and aggregates as a separate Edit. Update the in-body `**Aggregate:**` / `**Last refreshed:**` / `**Trend vs. prior:**` lines as another Edit. Update the History table at the bottom with a new row referencing the snapshot.

## Propagate (1) — Project OS synced copy (Pattern A only)

For Pattern A engagements, after the canonical vault write succeeds:

1. **Target:** `~/Projects/{code}/{code}-project-os/docs/project/private/meddpicc-scorecard.md`
2. **Action:** Overwrite with the same content as the canonical vault file, with two adjustments:
   - **Frontmatter:** add `synced_from: "[vault]/10-Engagements/{code}/meddpicc.md"` and `synced_at: YYYY-MM-DD` (today's date — distinct from `last_updated` because the sync may run a different day than the scoring).
   - **Header callout:** replace the vault file's "Singular source of truth" callout with a "Synced copy" callout pointing back at the canonical vault path. The callout makes clear: *do not hand-edit this file — edit the vault and re-sync.*
   - **Relative paths:** in body text, rewrite relative paths that referenced vault locations (`notes/...`, `reference/...`, `README.md`) to absolute paths or `[vault]/10-Engagements/{code}/...` form so the Project OS file is self-readable.
3. **Tool:** `Write` (full-file overwrite). The Project OS file is wholly derived from the vault scorecard — partial Edits don't make sense.
4. **No git commit.** Harper reviews.

If the Project OS repo doesn't exist for this engagement (Pattern A is declared in README but the repo is missing), surface that to Harper rather than failing silently.

## Propagate (2) — Vault README YAML

The vault's engagement README at `[vault]/10-Engagements/{code}/README.md` carries `meddpicc_tally` and `meddpicc_updated` frontmatter fields that feed the **Active Engagements** table on `00-System/HOME.md`. These are denormalized for dashboard rendering.

### Update the YAML fields

Read the README's frontmatter (between the leading `---` lines). Update or insert two fields:

```yaml
meddpicc_tally: "N 🟢 / N 🟡 / N 🟥"     # quote string — emoji + slashes break unquoted YAML
meddpicc_updated: YYYY-MM-DD             # ISO date, matches the scorecard's new last_updated
```

Use the `Edit` tool with a targeted old_string → new_string. Do not rewrite the whole file. Other frontmatter fields (`engagement_code`, `client`, `status`, `stage`, `pattern`, `project_os_path`, `started`, `gm`, `last_reviewed`) must remain byte-identical.

If either field is missing entirely, add it just before the closing `---`. If both are present, do an in-place replacement.

### Tally formatting

- Three-color schema: `"N 🟢 / N 🟡 / N 🟥"` (emoji + space + slash + space, double-quoted)
- If the canonical scorecard carries a 4th category (⚪ unscored, used on some engagements like HP), **fold ⚪ counts into 🟥 for the README tally string**. Conservative read — unscored = blocking. The canonical scorecard keeps its 4-color schema; only the denormalized README string compresses to 3.
- Shadow engagements (`pattern: B (shadow)`) carry `meddpicc_tally: "— shadow —"`. Don't overwrite that string — if the engagement is shadow-only, this skill shouldn't be running for it at all.

### When this propagation step does NOT apply

- **The vault README doesn't exist** — surface to Harper. The engagement should have one.
- **The README has no existing frontmatter** — surface to Harper. Don't silently re-insert.

## Do not synthesize evidence that isn't there

If new memory doesn't contain evidence for a dimension, leave that dimension's row untouched. Don't promote based on inference — only based on quoted, sourced, attributed claims. The discipline of waiting for evidence is what makes the scorecard worth maintaining.

This is the most common failure mode: updating a dimension because "the conversation has moved on" without specific new evidence. Don't do that. If the dimension feels stale but no new evidence exists, the right answer is to leave it and flag it to Harper as a candidate for the next discovery question.

## Conflict and drift handling

If a memory artifact contradicts the scorecard's current state (e.g., scorecard says "William attended 5/7 silently" but a daily log says "William missed 5/7"), trust the most recent + most explicit source. Update the scorecard cell and add a brief note flagging the correction. Memory drift is real — when you find drift, fix it rather than perpetuating.

If two memory artifacts conflict with each other, surface that to Harper rather than guessing. Conflicting evidence is a signal worth her attention.

If the Project OS synced copy is older than the canonical vault file when this skill runs, that's expected — the propagation step will overwrite it. If the Project OS file is *newer* than the vault file (someone hand-edited the Project OS copy), **stop and surface to Harper** — that's a violation of the canonical-vs-synced rule and shouldn't be silently clobbered.

## Action-item extraction discipline

When reading action items or decisions to update the scorecard, do not translate "client surfaced X as a criterion" into "client requires deliverable Y from us." Client-side asks are not vendor-side deliverables. The scorecard tracks what was actually said, committed, or done — not inflated translations. (Harper's correction pattern from prior sessions; carrying it forward.)

## Delivery

After updating:
1. Show Harper a focused summary of what changed — which dimensions moved, what the new evidence was, what citations were added
2. Tell her the new aggregate vs. prior aggregate with the delta
3. Confirm all four artifacts were written:
   - Canonical vault scorecard at `[vault]/10-Engagements/{code}/meddpicc.md`
   - Snapshot at `[vault]/10-Engagements/{code}/reference/meddpicc-snapshots/{prior-date}.md`
   - Project OS synced copy at `~/Projects/{code}/{code}-project-os/docs/project/private/meddpicc-scorecard.md` (Pattern A only)
   - Vault README YAML (`meddpicc_tally`, `meddpicc_updated`)
4. Flag any dimension where you considered changing state but didn't have enough evidence — these are good candidates for "what we still need to learn" lists / next discovery questions
5. Surface any drift you found between the canonical file and other memory artifacts (project-state.md, action-items.md) — Harper may want to reconcile those separately
6. Don't auto-commit to git — Harper reviews first

## Why this skill exists

The scorecard's value is the trajectory: week-over-week movement is what tells you whether the deal is converging toward a decision or stalling. A scorecard that gets updated consistently — even with small changes — is far more useful than one that gets refreshed every six weeks. This skill makes the update step low-effort enough that it actually happens, and disciplined enough that the updates remain trustworthy.

## When NOT to use this skill

- No scorecard exists yet — invoke `meddpicc-scorecard-bootstrap` instead
- Harper is asking for ad-hoc qualification analysis (e.g., "where do we stand on E?") without wanting the file updated — answer her question directly using the existing scorecard as context, don't write
- A meaningful reframe has happened (e.g., use case completely changed, sponsor changed) — at that point the scorecard may need a partial bootstrap rather than a marginal update; ask Harper which she wants
- The engagement is shadow-only (`pattern: B (shadow)`) — Harper isn't the GM, so the scorecard isn't his to maintain
