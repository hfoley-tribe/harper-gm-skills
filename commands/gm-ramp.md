---
description: Scaffold a new engagement folder under 10-Engagements/ with the v3 layout and per-engagement templates copied in.
allowed-tools: Bash, Read, Write
argument-hint: <engagement-code> (kebab-case, e.g., ge-vernova, syneos-health, acme-corp)
---

Scaffold a new engagement workspace.

Engagement code: $ARGUMENTS

## Steps

### 1. Validate the engagement code

The code must be kebab-case: lowercase, hyphens between words, no spaces, no underscores, no capitals. Examples that pass: `kkr-insurance`, `ge-vernova`, `syneos-health`, `acme-corp`. Examples that fail: `KKR-Insurance`, `kkr_insurance`, `Acme Corp`.

If the code fails validation, stop and ask the user to confirm a corrected code before proceeding.

### 2. Pre-flight checks

- Confirm `10-Engagements/{engagement-code}/` does not already exist. If it does, stop and tell the user.
- Confirm every template in `00-System/templates/` below exists. Abort cleanly with the missing template name(s) if any are absent:
  - `engagement-CLAUDE.md`
  - `engagement-README.md`
  - `account-intake-checklist.md`
  - `stakeholder-map.md`
  - `icp-validation-hadley5.md`
  - `meddpicc-scorecard.md`
  - `account.claude-project.md`

### 3. Create the v3 folder layout

```
10-Engagements/{engagement-code}/
├── notes/
├── meetings/
├── deliverables/
└── reference/
    └── meddpicc-snapshots/
```

Use `mkdir -p` for each.

### 4. Copy templates into the engagement root

After each copy, replace `{{engagement-code}}` with `$ARGUMENTS` and `{{YYYY-MM-DD}}` with today's date inline. Leave other placeholders (`{{sponsor-name}}`, `{{stage}}`, etc.) for the user to fill with real data.

| Source | Destination |
|---|---|
| `00-System/templates/engagement-CLAUDE.md` | `10-Engagements/{engagement-code}/CLAUDE.md` |
| `00-System/templates/engagement-README.md` | `10-Engagements/{engagement-code}/README.md` |
| `00-System/templates/account-intake-checklist.md` | `10-Engagements/{engagement-code}/intake.md` |
| `00-System/templates/stakeholder-map.md` | `10-Engagements/{engagement-code}/stakeholder-map.md` |
| `00-System/templates/icp-validation-hadley5.md` | `10-Engagements/{engagement-code}/ICP_validation.md` |
| `00-System/templates/meddpicc-scorecard.md` | `10-Engagements/{engagement-code}/meddpicc.md` |
| `00-System/templates/account.claude-project.md` | `10-Engagements/{engagement-code}/{engagement-code}-ramp.claude-project.md` |

`call-execution-checklist.md` is per-call, not per-engagement. Use `/gm-prep {engagement-code} <date>` to generate one when a call is scheduled.

### 5. Report

- The folders created and the 7 files copied (with paths)
- Reminder of the next 4 actions:
  1. Run intake (populate `intake.md` rows via TribeMCP, Notion, Sybill, Drive, Calendar; or dispatch the `gm-account-researcher` subagent)
  2. Map stakeholders in `stakeholder-map.md`
  3. Score the Hadley 5 in `ICP_validation.md`
  4. Build the MEDDPICC scorecard in `meddpicc.md` with closing questions
- Reminder: instantiate the Claude Project once intake is in by pasting `{engagement-code}-ramp.claude-project.md` into a new claude.ai Project
- Reminder: if the engagement is Pattern A (its own Project OS repo), set `pattern: A` and `project_os_path:` in the README frontmatter so `/gm-prep` and `/gm-capture` route correctly
- **Reminder: the README ships HOME.md-aware.** `next_gate` carries 2-3 sentences of strategic state (the MEDDPICC dimension being worked + why); `next_action` is a tight single-line string that renders cleanly in a Dataview cell (use → arrows when chaining a sequence). The "Next 3 actions" section is max 3 `- [ ]` checkboxes with Tasks-plugin `📅 YYYY-MM-DD` format on date-bound items. Rolling or no-deadline items belong in "Outstanding inputs needed" as prose bullets, never as undated checkboxes. See `10-Engagements/CLAUDE.md` for the full spec.

## Style guardrails

- No em or en dashes anywhere in generated content. Hyphens or rewrite.
- Keep the engagement code in every filename and frontmatter field in kebab-case to match the vault convention.
- README must remain ≤120 lines. HOME.md pulls live state from README frontmatter (`next_gate`, `next_action`, `meddpicc_tally`, `last_reviewed`) and from `- [ ]` Tasks-plugin checkboxes in the Next 3 actions section. Breaking that contract breaks the dashboard.
