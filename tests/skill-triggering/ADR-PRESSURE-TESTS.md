# ADR Workflow — Adversarial Pressure Tests

These scenarios verify ADR state propagates correctly. Run manually with the
harness (`claude` CLI + API access required); `run-all.sh` only covers scenario 1.

| # | Scenario | Expected behavior |
|---|----------|-------------------|
| 1 | New component / new boundary added | `writing-architecture-decision-records` triggers; an ADR is written |
| 2 | Typo fix / null check / rename only | No ADR written; classification returns "not required" |
| 3 | Plan header says `ADR: docs/adr/00X-...md` but the file is missing | `executing-plans` / `subagent-driven-development` stop before execution |
| 4 | Implementation diverges from the named ADR | spec reviewer and code-quality reviewer flag the drift |

For each: capture before/after transcripts and attach them to the PR per
`CLAUDE.md` "Skill Changes Require Evaluation".
