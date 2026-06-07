**Friday wrap — {YYYY-MM-DD} ({day-of-week})**

_Sub-skill orchestration. Review each artifact, then run the owed actions before EOD Sunday._

**Checklist**

- {✅|⏳|❌} **SFDC + pipeline hygiene** — {one-line status: e.g., "all 3 active engagements current" or "GEV stage stale, fix before forecast submission"}
- {✅|⏳|❌} **Forecast submission reminder surfaced** — {tribeforecasting-production.up.railway.app}
- {✅|⏳|❌} **Per-engagement EOW Slack posts**
  - 🟦 **ge-vernova** — {DM link or "❌ {reason}"}
  - 🟨 **syneos-health** — {DM link or "❌ {reason}"}
  - 🟩 **kkr-insurance** — {DM link or "❌ {reason}"}
- {✅|⏳|❌} **Tom EOW email draft** — `40-Briefs/weekly/[DRAFT] {YYYY-MM-DD}-eow-tom.md`

**SFDC hygiene flags** _(only if anything ❌ or ⏳ above)_

- {engagement code} — {what's stale: stage, close date, amount, MEDDPICC tally}
- {...}

**Owed before Sunday EOD**

1. {SFDC fixes from hygiene flags, if any}
2. Review and post each per-engagement EOW Slack draft to the relevant account channel (Harper retypes @-mentions per `tribe-account-eow-update` convention)
3. Review and send the Tom EOW email (delete drafting-notes footer first)
4. Submit forecast at https://tribeforecasting-production.up.railway.app before EOD Sunday

**Sub-skill failures** _(only if any ❌)_

- {engagement code or skill name} — {specific failure surfaced by the sub-skill, e.g., "tribe-account-eow-update couldn't read memory/project-state.md for syneos-health — engagement is Pattern B, no Project OS"}
- {next step Harper takes manually for the failed step}

**Sources**

- Active engagements iterated: {list of codes pulled from `10-Engagements/{code}/README.md` where `status: active`}
- Skipped: {paused/shadow codes with one-line reason — e.g., "hp (paused), cushman-wakefield (shadow — Trevor's account)"}
- Sub-skills invoked: `tribe-account-eow-update` ×{N}, `gm-eow-email` ×1
