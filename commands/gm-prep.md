---
description: Generate a filled-in pre-call execution checklist for an upcoming engagement call.
allowed-tools: Read, Write, Glob, Grep
argument-hint: <engagement-code> <YYYY-MM-DD> (e.g., kkr-insurance 2026-05-12)
---

Generate a pre-call execution checklist.

Arguments: $ARGUMENTS

Parse: first token = engagement code (kebab-case, e.g. `kkr-insurance`, `ge-vernova`); second token = call date in `YYYY-MM-DD` format.

## Steps

### 1. Confirm the engagement exists

Check `10-Engagements/{engagement-code}/`. If missing, stop and tell the user to run `/gm-ramp {engagement-code}` first.

### 2. Read the engagement context (vault, both patterns)

Skip and note any that are missing:

- `10-Engagements/{engagement-code}/README.md` (current state, posture, stakeholders)
- `10-Engagements/{engagement-code}/meddpicc.md` (present-state, closing questions, Hadley filter, adversarial questions)
- `10-Engagements/{engagement-code}/stakeholder-map.md` (sponsor, IC, peers)
- `10-Engagements/{engagement-code}/ICP_validation.md` (Hadley 5 disqualifier scores)
- `10-Engagements/{engagement-code}/intake.md` (recent state, history)
- Any `*Brief*.md` file in the engagement folder (use Glob)

### 3. If Pattern A, also read the Project OS

Check the engagement README's frontmatter for `pattern: A` and `project_os_path`. If Pattern A:

- `~/Projects/{engagement-code}/{engagement-code}-project-os/memory/project-state.md` (current state of play)
- The latest 2 to 3 files in `~/Projects/{engagement-code}/{engagement-code}-project-os/memory/daily/` (most recent operational signal)
- `~/Projects/{engagement-code}/{engagement-code}-project-os/memory/action-items.md` (open commitments)

The current Pattern A list is just `kkr-insurance`. Pattern detection should still come from the README, not a hard-coded list.

### 4. Read cross-cutting frameworks

- `00-System/templates/call-execution-checklist.md` (template structure)
- `15-Onboarding/gm-insights-v1.md` (Hadley 5, MetLife technique, reference customers)
- `35-Sales-Reference/hadley-5-disqualifiers.md`
- `35-Sales-Reference/meddpicc-method.md`
- `35-Sales-Reference/archetype-composition.md`
- `15-Onboarding/scenario-log.md` (relevant prior scenarios)

### 5. Generate a filled-in call execution checklist

- **Pre-call section:** pull pending action items from the MEDDPICC scorecard's "Gap to close" column.
- **0 to 10 min, domain framing:** 4 to 6 beats based on the engagement's industry and business model. Establish peer-level fluency.
- **10 to 35 min, discovery questions:** pull "closing questions" from the MEDDPICC scorecard.
- **10 to 35 min, qualification questions:** pull from Hadley 5; any 🟥 or 🟡 row's evidence-gap question.
- **35 to 55 min, closer:** based on engagement posture (partnership-or-walk for KKR, expansion for GEV, productionization for technical buyers).
- **Reference customer order:** pull from the README's posture or weekly-update sections.
- **55 to 60 min, commitment ask:** pull from MEDDPICC's "next concrete move."
- **🟥 Sponsorship test:** pull from ICP_validation 🟥 markers and stakeholder map gaps.
- **Walk-away conditions:** pull from MEDDPICC + ICP scorecard 🟥 markers + posture-defined "what makes the math fail."
- **Post-call commitments:** standard template plus any engagement-specific items.

### 6. Save the checklist

Save to: `10-Engagements/{engagement-code}/meetings/{YYYY-MM-DD}-call-execution-checklist.md`.

For Pattern A engagements, also drop a copy to `~/Projects/{engagement-code}/{engagement-code}-project-os/memory/calltranscripts/{YYYY-MM-DD}-call-prep.md` so the prep sits next to the eventual transcript.

### 7. Report

- The file or files created
- Sections that could not auto-fill (so the user can complete them manually)
- Adversarial questions worth surfacing pre-call (from MEDDPICC's "3 adversarial questions" block)
- Reminder: read walk-away conditions aloud once before the call (lock the commitment)

Be concrete. Quote evidence from source files inline. The output should be printable and walk-into-the-room ready.

## Style guardrails

- No em or en dashes. Hyphens or rewrite.
- Verify stakeholder name spellings against the source before writing them into the checklist.
- If a section relies on inference rather than sourced material, mark it `(inferred)` so Harper can refine in the room.
