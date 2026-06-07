---
description: Post-call capture. Transcript or Granola-synced meeting note becomes MEDDPICC deltas, stakeholder updates, a scenario-log entry, and a Salesforce paste block.
allowed-tools: Read, Write, Edit, Glob, mcp__claude_ai_TribeMCP__pull_transcript, mcp__claude_ai_TribeMCP__search_transcripts, mcp__plugin_Notion_notion__notion-query-meeting-notes, mcp__plugin_Notion_notion__notion-fetch, mcp__claude_ai_Google_Calendar__list_events, mcp__claude_ai_Google_Calendar__get_event
argument-hint: <engagement-code> <transcript-or-note-id> (e.g., kkr-insurance 2026-05-07-neir-followup)
---

Run post-call capture. Adapted from Brandon's Granola to Ashby pattern, with Pattern A vs. Pattern B routing.

Arguments: $ARGUMENTS

Parse: first token = engagement code (kebab-case, e.g. `kkr-insurance`, `ge-vernova`); second token = transcript or meeting-note identifier (UUID, Notion page id, or descriptive slug like `2026-05-07-neir-followup`).

## Steps

### 1. Resolve engagement pattern

Read `10-Engagements/{engagement-code}/README.md` frontmatter. Look for `pattern:` (A or B) and `project_os_path:`.

- **Pattern A** (Project OS engagement, currently `kkr-insurance`): write outputs into the Project OS repo at the path in `project_os_path`.
- **Pattern B** (everything else): write outputs into the vault.

If `pattern` is missing, default to B and note it in the final report.

### 2. Pull the source material

Try in this order until you have a verbatim transcript or a Granola-quality meeting note:

1. `mcp__claude_ai_TribeMCP__pull_transcript` with the identifier if it looks like a Sybill / TribeMCP transcript id.
2. `mcp__claude_ai_TribeMCP__search_transcripts` with the engagement code + a date hint if the id is a slug.
3. `mcp__plugin_Notion_notion__notion-query-meeting-notes` for the engagement keyword (Granola syncs into Notion; Granola-quality notes live there). Then `mcp__plugin_Notion_notion__notion-fetch` to read the matched page.
4. `mcp__claude_ai_Google_Calendar__list_events` or `get_event` for missing metadata (date, attendees, duration).

Record: date, attendees, duration, source URL or id.

### 3. Read the engagement's current state

- `10-Engagements/{engagement-code}/meddpicc.md` (canonical vault filename per `project_meddpicc_dual_file` memory)
- `10-Engagements/{engagement-code}/stakeholder-map.md` if present
- `10-Engagements/{engagement-code}/README.md` for posture and recent status
- For Pattern A: also read `~/Projects/{engagement-code}/{engagement-code}-project-os/memory/project-state.md` and `memory/action-items.md` for current state.

### 4. Generate four outputs into one capture file

**A. Salesforce activity log paste block**

```
Date: {date}
Attendees: {list}
Duration: {duration}

Summary:
- {bullet 1}
- {bullet 2}
- {bullet 3}

Action items:
- [@owner] {task} {due date}
```

**B. MEDDPICC scorecard deltas** for each dimension that changed:

- **{Dimension}**
  - BEFORE: {present-state from current scorecard}
  - AFTER: {updated present-state from transcript}
  - EVIDENCE: > "{verbatim quote}"

Apply the pain discipline: pain is GREEN only when a named customer stakeholder articulates specific pain (per `feedback_meddpicc_pain_scoring`). Industry-level awareness alone is YELLOW.

**C. Stakeholder map updates**

- **+ Added:** {name, title, role, evidence}
- **Updated:** {name, change, evidence}

Verify name spellings against the source verbatim before writing. Do not invent titles. If a role is inferred rather than stated, mark it `(inferred)`.

**D. Scenario log entry** (only if a NEW scenario type surfaced)

Check `15-Onboarding/scenario-log.md` first. Skip if this scenario already has an entry.

- **N. {Scenario name}**
  - Trigger:
  - Decision-maker:
  - Data needed:
  - Escalation rule:
  - Source: This call ({id}, {date})

### 5. Write the capture file

- **Pattern A:** `~/Projects/{engagement-code}/{engagement-code}-project-os/memory/calltranscripts/{YYYY-MM-DD}-{slug}.md`
- **Pattern B:** `10-Engagements/{engagement-code}/meetings/{YYYY-MM-DD}-{slug}.md`

Slug = short kebab-case description (e.g., `neir-followup`, `pete-collab-review`).

### 6. Append scenario log if D non-empty

Append section D to `15-Onboarding/scenario-log.md` with the next sequential entry number.

### 7. Report back

- Capture file path
- "Section A ready for Salesforce paste"
- Section B: MEDDPICC dimensions that need a manual scorecard refresh (or suggest invoking the `meddpicc-scorecard-update` skill)
- Section C: stakeholder map updates pending
- Section D: either "new scenario added: scenario-log.md entry #N" or "no new scenario type"
- Pattern detected (A or B) and the path used

## Style guardrails

- No em or en dashes. Hyphens or rewrite.
- Quote evidence verbatim. Do not paraphrase a stakeholder into a sharper version of what they said.
- If attribution is uncertain (who said what, who owns an action item), surface the uncertainty rather than guess.
