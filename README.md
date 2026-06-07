# Harper GM Skills

Extracted GM workflow skills and slash commands from Harper Foley's Tribe AI setup.

Source:
- Vault skill folder: `.claude/skills/`
- Higher-level skill folder: `~/.claude/skills/`
- Higher-level command folder: `~/.claude/commands/`
- Inventory reference checked: `00-System/skills-reference.md`
- Extracted on: 2026-06-07

## Included Skills

| Skill | Purpose |
|---|---|
| `gm-cadence-prep` | Standing weekly cadence prep for Sunday, Monday, Tuesday, and Friday modes. Includes vault prep templates and Slack DM templates. |
| `gm-calendar-event` | Creates Google Calendar events in Harper's bracketed title plus six-section HTML description format. |
| `gm-daily-brief` | Generates Harper's daily GM brief from calendar, Slack, Gmail, calls, notes, engagement READMEs, and Project OS context. |
| `gm-eow-email` | Generates Harper's Friday end-of-week email draft to Tom Lee. Includes the EOW email template. |
| `meddpicc-scorecard-bootstrap` | Creates the initial MEDDPICC scorecard for a Tribe AI client account. |
| `meddpicc-scorecard-update` | Refreshes an existing MEDDPICC scorecard, snapshots prior state, and syncs dashboard fields. |
| `tribe-account-eow-update` | Generates account-specific Friday EOW Slack update previews. |

The full skill directories are under `skills/`, preserving each `SKILL.md` and its templates.

## Included Slash Commands

| Command | Purpose |
|---|---|
| `/gm-ramp` | Scaffolds a new engagement workspace from the v3 templates. |
| `/gm-prep` | Generates a pre-call execution checklist for an upcoming engagement call. |
| `/gm-capture` | Runs post-call capture from transcript or meeting note into MEDDPICC deltas, stakeholder updates, scenario log entry, and Salesforce paste block. |

Command definitions are under `commands/`, preserving their Claude Code command markdown format.

## Install

For Claude Code-style local skills:

```bash
cp -R skills/* ~/.claude/skills/
cp commands/gm-*.md ~/.claude/commands/
```

For Codex-style local skills, copy the skill directories into the configured Codex skills directory instead.

## Notes

- The vault reference also mentions `gm-account-researcher`, which is an agent under `~/.claude/agents/`, not a skill or slash command. It is not included in this repo snapshot.
- Utility skills such as `defuddle`, `obsidian-markdown`, `obsidian-bases`, `obsidian-cli`, and `json-canvas` were listed in the vault reference but are external or installed skills, not Harper GM skill directories in `.claude/skills/`.
- `gm-calendar-event` intentionally contains Harper-specific local vault `file:///Users/harperfoley/...` link conventions. Adjust those if using the skill on another machine.
- Several cadence and EOW skills intentionally contain Harper-specific Slack DM IDs. Adjust those if installing for another user.
