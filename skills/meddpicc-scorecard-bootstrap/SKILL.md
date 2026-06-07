---
name: meddpicc-scorecard-bootstrap
description: Create the initial MEDDPICC scorecard for a Tribe AI client account. Synthesizes a v0 scorecard from existing project memory (account brief, harper-notes-on-understanding, daily logs, decisions, call transcripts) using Harper's canonical format — RAG colors (🟢🟡🟥), M/E/D/D/P/I/C/C order, field-by-field table with evidence + conversion criteria, optional Hadley 5-disqualifier filter, optional adversarial review questions. Writes the canonical vault file at `[vault]/10-Engagements/{code}/meddpicc.md`, then for Pattern A engagements propagates a synced copy to `~/Projects/{code}/{code}-project-os/docs/project/private/meddpicc-scorecard.md` and seeds the vault README's `meddpicc_tally` + `meddpicc_updated` frontmatter so the dashboard at `00-System/HOME.md` picks the engagement up. Use whenever Harper asks to "set up MEDDPICC", "create a scorecard", "initialize MEDDPICC tracking", "bootstrap MEDDPICC", "score [account] from scratch", "start MEDDPICC for [account]", or any equivalent for starting MEDDPICC tracking on a new account. Use even if Harper doesn't explicitly say "scorecard" — if she's asking to qualify a deal in MEDDPICC structure for the first time, this is the right tool.
---

# MEDDPICC Scorecard Bootstrap

Create Harper's initial MEDDPICC scorecard for a new Tribe AI account. The scorecard is **GM-layer scoring** and lives in the vault as the canonical artifact, with optional propagation to a Project OS synced copy for Pattern A engagements. Pairs with `meddpicc-scorecard-update` (which refreshes existing scorecards).

## Canonical-vs-synced architecture

> **Read this before touching any path.**

| File | Role | Pattern A | Pattern B |
|---|---|---|---|
| `[vault]/10-Engagements/{code}/meddpicc.md` | **Canonical.** Single source of truth. Bootstrap writes here first. | ✅ | ✅ |
| `~/Projects/{code}/{code}-project-os/docs/project/private/meddpicc-scorecard.md` | **Synced copy** of the vault scorecard. Propagated for in-repo visibility. | ✅ | n/a |
| `[vault]/10-Engagements/{code}/README.md` frontmatter (`meddpicc_tally`, `meddpicc_updated`) | **Denormalized** for the dashboard. | ✅ | ✅ |

**Rule:** the vault file is canonical. The Project OS file is wholly derived from it.

## Resolve `{code}` first

The engagement code drives every path. Resolve in this order:

1. **From an explicit arg** if Harper said "bootstrap MEDDPICC for kkr-insurance" — use that string verbatim.
2. **From CWD if running inside a Project OS repo** (`~/Projects/{code}/{code}-project-os/`):
   ```bash
   CODE=$(basename "$PWD" | sed 's/-project-os$//')
   ```
3. **From CWD if running inside the vault engagement folder** (`[vault]/10-Engagements/{code}/`):
   ```bash
   CODE=$(basename "$PWD")
   ```
4. **Otherwise, ask** rather than guess.

Sanity-check: `{code}` should match an existing folder under `[vault]/10-Engagements/`. If the engagement folder doesn't exist yet, the bootstrap should ask Harper to scaffold it first (`mkdir -p [vault]/10-Engagements/{code}/`) — the README and CLAUDE.md need to exist before the scorecard depends on them.

## Read pattern from vault README

Look up `[vault]/10-Engagements/{code}/README.md` frontmatter `pattern:` field. Drives memory inputs and whether to propagate to Project OS:

- `pattern: A` → Project OS exists; read memory from Project OS; propagate to Project OS scorecard
- `pattern: B` → No Project OS; read memory from vault; no Project OS propagation
- `pattern: B (shadow)` → Don't bootstrap. Surface to Harper — Harper isn't the GM, scorecard isn't his to maintain.

## What MEDDPICC is (briefly)

A B2B sales qualification framework. Each letter is a dimension that needs a buyer-validated answer for a deal to progress:

- **M — Metrics** — quantified business value (buyer-owned, not vendor-claimed)
- **E — Economic Buyer** — who has authority to release budget; sign-off path
- **D — Decision Criteria** — what criteria the buyer uses to evaluate options
- **D — Decision Process** — what steps lead to a yes/no, in what order, on what timeline
- **P — Paper Process** — legal, security, procurement, MSA cycle
- **I — Identified Pain** — what hurts, who feels it, why it can't continue
- **C — Champion** — internal advocate willing to spend political capital (distinct from sponsor)
- **C — Competition** — alternatives being evaluated (including build, do-nothing, incumbent)

A dimension is "answered" when there's specific, sourced evidence — a quote, a name, a date, a number — not when there's plausible inference.

## Inputs to read

Pull evidence directly from these artifacts. Don't paraphrase early; the scorecard wants verbatim quotes with citations.

### Pattern A (Project OS exists)

| Artifact | Path | Used for |
|---|---|---|
| Engagement CLAUDE.md | `[vault]/10-Engagements/{code}/CLAUDE.md` | Account name, sponsor, watch-outs, current priorities, visibility flag |
| Engagement README | `[vault]/10-Engagements/{code}/README.md` | Current state, stakeholders, workstreams |
| Project OS CLAUDE.md | `~/Projects/{code}/{code}-project-os/CLAUDE.md` | Visibility flag, delivery posture |
| Account brief | `~/Projects/{code}/{code}-project-os/docs/project/private/account-brief-*.md` | Use case, stakeholders, commercial status, known constraints |
| Harper notes | `~/Projects/{code}/{code}-project-os/docs/project/private/harper-notes-on-understanding.md` | Domain understanding, hypothesized opportunity, open questions |
| Daily logs | `~/Projects/{code}/{code}-project-os/memory/daily/*.md` | Specific quotes, attendance, timestamped events |
| Decisions log | `~/Projects/{code}/{code}-project-os/memory/decisions.md` | Anything ratified |
| Call transcripts | `~/Projects/{code}/{code}-project-os/memory/calltranscripts/*.md` | Verbatim evidence with timestamps and speaker attribution |
| Action items | `~/Projects/{code}/{code}-project-os/memory/action-items.md` | Open work surfaced from calls |
| GM notes | `[vault]/10-Engagements/{code}/notes/*.md` | Harper's private GM thinking, political reads |

### Pattern B (vault-only)

| Artifact | Path | Used for |
|---|---|---|
| Engagement CLAUDE.md | `[vault]/10-Engagements/{code}/CLAUDE.md` | Account name, sponsor, watch-outs, voice |
| Engagement README | `[vault]/10-Engagements/{code}/README.md` | Current state, stakeholders, workstreams |
| Meeting synthesis | `[vault]/10-Engagements/{code}/meetings/*.md` | Recent meetings, decisions, action items |
| GM notes | `[vault]/10-Engagements/{code}/notes/*.md` | Harper's private GM thinking, political reads |
| Deliverable drafts | `[vault]/10-Engagements/{code}/deliverables/*.md` | What's been sent, what's in flight |
| Live MCPs | Sybill, Granola, Slack | Pull on demand for evidence not yet in the vault |

If any of these are missing, that's fine — read what's there. If almost nothing exists yet, the bootstrap will mostly be 🟥s, which is the correct state for a brand-new account.

## Output: scorecard structure

Write to `[vault]/10-Engagements/{code}/meddpicc.md`. Use this structure:

```markdown
---
engagement: {code}
artifact: meddpicc-current
last_updated: YYYY-MM-DD
scored_by: Harper Foley
stage: [qualifying | discovery | proposal | negotiation | etc. — match README stage]
aggregate_green: N
aggregate_yellow: N
aggregate_red: N
prior_snapshot_date: null
prior_aggregate: null
tags: [engagement, meddpicc, {code}]
---

# MEDDPICC — [Client Name] (current)

> **Singular source of truth for current MEDDPICC scoring on this account. Overwrite on refresh; do not append.**
> Frozen prior snapshots for trend analysis live in [reference/meddpicc-snapshots/](reference/meddpicc-snapshots/).
> [If Pattern A:] Pattern A engagement — operational source of truth lives in Project OS at `~/Projects/{code}/{code}-project-os/`. **MEDDPICC is GM-layer scoring and lives in this vault, not in Project OS.** Synced copy is propagated to Project OS for in-repo visibility.

**Stage:** [match stage]
**Aggregate:** N 🟩 / N 🟨 / N 🟥
**Last refreshed:** YYYY-MM-DD (Harper Foley) — initial bootstrap
**Trend vs. prior:** First field-by-field capture.

> **Important framing (optional):** [Any active reframe or watch-out the reader needs to know before reading the field rows.]

---

## Field-by-field

| Field | Status | Score / Value | Evidence | Converts to GREEN when |
|---|---|---|---|---|
| **M — Metrics** | [🟩/🟨/🟥] | [quoted value or "TBD — unknown"] | [Sybill UUID + timestamp, Project OS memory path, or "no evidence captured"] | [the cheapest concrete piece of evidence that would move it to 🟩] |
| **E — Economic Buyer** | ... | ... | ... | ... |
| **D — Decision Criteria** | ... | ... | ... | ... |
| **D — Decision Process** | ... | ... | ... | ... |
| **P — Paper Process** | ... | ... | ... | ... |
| **I — Identify Pain** | ... | ... | ... | ... |
| **C — Champion** | ... | ... | ... | ... |
| **C — Competition** | ... | ... | ... | ... |

---

## What converts the deal forward

[1–3 paragraphs naming the single event or evidence-set that would move multiple fields at once — e.g., a specific upcoming call, a deliverable, a decision. This makes the scorecard actionable.]

---

## Refresh triggers

Re-score and overwrite this file after any of:
- [Concrete trigger 1 — usually an upcoming meeting or deliverable]
- [Concrete trigger 2 — material stakeholder event]
- [Cadence-drift disqualifier if relevant]
- Monthly cadence at minimum, even if no triggering event

When refreshing: copy the prior file to `reference/meddpicc-snapshots/YYYY-MM-DD.md` first (frozen history), then overwrite this file. The `meddpicc-scorecard-update` skill handles this automatically.

---

## History

| Date | Aggregate | Source | Notes |
|---|---|---|---|
| YYYY-MM-DD | NG / NY / NR | _current_ | Initial bootstrap. |

---

## Related

- [README.md](README.md) — engagement state, people, workstreams.
- [CLAUDE.md](CLAUDE.md) — operating posture, watch-outs.
- [List of key notes / meeting synthesis files relevant to scoring]
- [If Pattern A:] Project OS `~/Projects/{code}/{code}-project-os/memory/` — operational source of truth for delivery; this scorecard pulls evidence from there.
```

### Optional sections

The current canonical format (per `kkr-insurance/meddpicc.md`) does NOT include the Hadley 5-disqualifier filter or adversarial review questions at the scorecard level. Both are useful patterns, but Harper may or may not want them on a given account.

- **Include them on bootstrap only if** Harper has explicitly asked, or if the account is at a stage where qualification rigor matters (early discovery / late-stage proposal).
- **Otherwise skip** — they can be added on a later refresh if Harper wants them.

If included, the Hadley filter and adversarial questions go below the History section, before Related. (See the legacy bootstrap format for templates.)

## Filling in the scorecard

### Default to 🟥 unless evidence supports otherwise

When the project memory doesn't contain specific, quoted evidence for a dimension, mark it 🟥 with "TBD — unknown" or "Unanswered" in the Score/Value cell. Do not guess. The point of the scorecard is to surface what's not yet known — over-claiming early creates false confidence and obscures real discovery work.

A dimension goes 🟨 only if there's partial information with a clear path to confirmation. A dimension goes 🟩 only if there's a quoted, sourced, attributed claim that definitively answers it.

### Use direct quotes with citations in the Evidence column

Every Evidence cell that contains a claim cites its primary source. Format examples:
- Call transcript: `Sybill <UUID> [11:34] — Neir: *"100 bps... is probably very aggressive."*`
- Daily log: `Project OS memory/daily/2026-05-07-harper.md`
- Note: `Vault notes/2026-05-07-call-debrief.md`
- Email: `Gmail thread <thread-id>`

This is how reviewers trust the scorecard — every claim is traceable to a primary source in 2 clicks.

### Sponsor vs. Champion (commonly conflated)

The sponsor is whoever brought the conversation in (often someone with title authority). The champion is whoever spends political capital internally to see Tribe win — knows the buying process, names other stakeholders, helps Tribe navigate land mines, takes hits when things go sideways. Most early-stage deals have a sponsor (🟨) but no champion (🟥). Do not auto-promote the sponsor into the champion slot at bootstrap; that's a load-bearing distinction.

### Identify Pain scoring discipline (per memory `feedback_meddpicc_pain_scoring`)

Pain is 🟩 **only when customer-specific named pain is captured** — which workflow, which KPI, which constraint blocks today. Industry-level pain awareness ("the sector has this problem; the customer acknowledged it") is 🟨. The customer ratifying the *industry* frame is not the same as the customer naming *their own* pain.

### "Converts to GREEN when" column

This column is what makes the scorecard actionable. For each dimension, name the **cheapest concrete piece of evidence** that would move it up to 🟩. Specific: "Functional leader surfaces a quantified outcome — basis-point cost-of-funds delta or sourcing throughput target, anchored to a dollar number Neir's team underwrites." Not vague: "more discovery."

## Snapshot directory setup

Create the snapshot directory at bootstrap time so the `meddpicc-scorecard-update` skill can write into it without checking existence:

```bash
mkdir -p [vault]/10-Engagements/{code}/reference/meddpicc-snapshots
```

## Write the canonical vault file

Use the `Write` tool (full file). After writing, **read back** the file and confirm the YAML frontmatter parses cleanly (date fields are unquoted ISO, emoji-bearing strings are double-quoted, no smart quotes).

## Propagate (1) — Project OS synced copy (Pattern A only)

For Pattern A engagements only:

1. **Target:** `~/Projects/{code}/{code}-project-os/docs/project/private/meddpicc-scorecard.md`
2. **Action:** Write the same content as the canonical vault file, with adjustments:
   - **Frontmatter add:** `synced_from: "[vault]/10-Engagements/{code}/meddpicc.md"`, `synced_at: YYYY-MM-DD` (today's date)
   - **Header callout:** replace the vault file's "Singular source of truth" callout with a "Synced copy" callout pointing back at the canonical vault path. Make clear: *do not hand-edit this file — edit the vault and re-sync.*
   - **Relative paths:** rewrite relative paths that referenced vault locations to absolute or `[vault]/...` form so the Project OS file is self-readable.
3. **Tool:** `Write` (full-file overwrite).
4. **If `docs/project/private/` doesn't exist in Project OS,** create it: `mkdir -p ~/Projects/{code}/{code}-project-os/docs/project/private`.
5. **No git commit.** Harper reviews.

## Propagate (2) — Vault README YAML

Update `[vault]/10-Engagements/{code}/README.md` frontmatter:

```yaml
meddpicc_tally: "N 🟢 / N 🟡 / N 🟥"     # quote string — emoji + slashes break unquoted YAML
meddpicc_updated: YYYY-MM-DD             # today's date
```

Use the `Edit` tool with a targeted old_string → new_string. Do not rewrite the whole file. Other frontmatter fields must remain byte-identical.

If either field is missing, insert just before the closing `---`. If present (from a prior bootstrap or from the dashboard's initial seeding), do an in-place replacement.

For 4-color scorecards with a ⚪ category, fold ⚪ counts into 🟥 for the README's 3-color tally string. Canonical scorecard keeps the 4 colors; only the denormalized string compresses.

## Visibility check

Read the engagement's `CLAUDE.md` for the visibility flag. The vault is Harper-personal, so financial figures are fine in the canonical scorecard. **The Project OS synced copy has the same visibility constraints as the rest of the Project OS repo.** If Project OS visibility is `tribe-shared`, the synced copy of the scorecard is visible to the team — that's actually the point of the sync. Harper has made the call to share scorecard contents with the team in that case.

If the Project OS visibility is `tribe-shared` AND the scorecard contains material that should stay GM-only (e.g., sensitive political reads about specific stakeholders), surface that to Harper before propagating — she may want to redact specific cells in the synced copy.

## Delivery

After writing:
1. Tell Harper the vault file path (canonical)
2. Confirm Project OS synced copy was written (Pattern A only) — give the path
3. Confirm vault README YAML was updated — show the new `meddpicc_tally` and `meddpicc_updated` values
4. Surface the aggregate + the 🟥 dimensions explicitly so she sees the gaps
5. Flag any dimension where the bootstrap is operating on thin evidence — those are the ones she should sanity-check first
6. Don't auto-commit to git — Harper reviews first

## Why this skill exists

A consistent scorecard format makes deal-state legible across accounts and over time. The bootstrap step is where it's easy to over-claim ("we have a champion!") or under-claim ("everything is unknown"). The discipline of this skill — defaults to 🟥, requires quoted evidence, distinguishes sponsor from champion, separates industry pain from customer pain — guards against both failure modes.

## When NOT to use this skill

- A scorecard already exists at `[vault]/10-Engagements/{code}/meddpicc.md` — use `meddpicc-scorecard-update` instead
- Harper is asking for ad-hoc qualification analysis without committing to a maintained scorecard — answer her question directly without writing a file
- The account is too early-stage for MEDDPICC to be useful (no first call held, no use case scoped) — tell Harper that and ask if she still wants the scaffolding
- The engagement is shadow-only (`pattern: B (shadow)`) — Harper isn't the GM
