# Harper GM Skills

Extracted GM workflow skills from Harper Foley's Tribe AI vault.

Source:
- Vault folder: `.claude/skills/`
- Inventory reference checked: `00-System/skills-reference.md`
- Extracted on: 2026-06-07

## Included Skills

| Skill | Purpose |
|---|---|
| `gm-cadence-prep` | Standing weekly cadence prep for Sunday, Monday, Tuesday, and Friday modes. Includes vault prep templates and Slack DM templates. |
| `gm-calendar-event` | Creates Google Calendar events in Harper's bracketed title plus six-section HTML description format. |
| `gm-daily-brief` | Generates Harper's daily GM brief from calendar, Slack, Gmail, calls, notes, engagement READMEs, and Project OS context. |
| `gm-eow-email` | Generates Harper's Friday end-of-week email draft to Tom Lee. Includes the EOW email template. |

The full skill directories are under `skills/`, preserving each `SKILL.md` and its templates.

## Install

For Claude Code-style local skills:

```bash
cp -R skills/* ~/.claude/skills/
```

For Codex-style local skills, copy the skill directories into the configured Codex skills directory instead.

## Notes

- The vault reference also mentions slash commands, agents, and workflow surfaces that were not present as directories in `.claude/skills/` at extraction time, including `/gm-ramp`, `/gm-prep`, `/gm-capture`, `gm-account-researcher`, `tribe-account-eow-update`, `meddpicc-scorecard-bootstrap`, and `meddpicc-scorecard-update`.
- Utility skills such as `defuddle`, `obsidian-markdown`, `obsidian-bases`, `obsidian-cli`, and `json-canvas` were listed in the vault reference but are external or installed skills, not Harper GM skill directories in `.claude/skills/`.
- `gm-calendar-event` intentionally contains Harper-specific local vault `file:///Users/harperfoley/...` link conventions. Adjust those if using the skill on another machine.
