# ADR Workflow Propagation Eval Results

## Environment

| Harness | Harness version | Model | Model version/ID | Date |
|---------|-----------------|-------|------------------|------|
| Claude Code | 2.1.141 | Claude | default CLI model | 2026-05-14 |

## Scenario Results

| # | Scenario | Baseline result | After-change result | Pass? | Transcript/log |
|---|----------|-----------------|---------------------|-------|----------------|
| 1 | New component / new boundary added | Initial run failed because Claude Code 2.1.141 requires `--verbose` with `--output-format=stream-json`; harness was patched before behavior evaluation. | `writing-architecture-decision-records` triggered as `superpowers:writing-architecture-decision-records`. | yes | Original `claude -p` stream-json logs lived under `/tmp/superpowers-tests/` and were not persisted; re-run via `tests/skill-triggering/run-test.sh writing-architecture-decision-records prompts/writing-architecture-decision-records.txt 3` and attach the resulting `claude-output.json` (or the relevant transcript excerpt) to the PR description. |
| 2 | Typo fix / null check / rename only | Static baseline from reviewed skill state: ADR table already classified typo/rename as not required; this run verifies the new propagation work did not over-trigger. | ADR skill did not trigger; assistant classified typo and local rename as cosmetic/local cleanup and said ADR is unnecessary. | yes | Manual run; transcript was not persisted. Re-run the scenario per `tests/skill-triggering/ADR-PRESSURE-TESTS.md` row 2 and attach the transcript excerpt to the PR. |
| 3 | Plan header names missing ADR file | Static baseline from reviewed execution skills: pre-change flow classified before reading the plan, so the missing `**ADR:**` field was not load-bearing. | `executing-plans` and `writing-architecture-decision-records` triggered; execution stopped before TodoWrite/task execution; `marker.txt` was not created; missing `docs/adr/999-missing-adr.md` was reported. | yes | Manual run; transcript was not persisted. Re-run the scenario per `tests/skill-triggering/ADR-PRESSURE-TESTS.md` row 3 and attach the transcript excerpt to the PR. |
| 4 | Implementation diverges from named ADR | Static baseline from reviewed `code-reviewer.md`: direct code review had no ADR field or ADR alignment checklist, so drift was not a first-class review concern. | `requesting-code-review` triggered and reported Critical ADR drift: code switched from JSON file storage to SQLite, which ADR-001 explicitly rejected. | yes | Manual run; transcript was not persisted. Re-run the scenario per `tests/skill-triggering/ADR-PRESSURE-TESTS.md` row 4 and attach the transcript excerpt to the PR. |

## Notes

- Scenario 1 is covered by `tests/skill-triggering/run-test.sh`.
- Scenarios 2-4 were run manually and recorded above.
- If any scenario fails, tune the relevant skill/prompt and re-run that scenario before completing the branch.
- The Scenario 1 baseline is a harness compatibility failure, not an ADR behavior failure. The fix adds `--verbose` to `tests/skill-triggering/run-test.sh` so current Claude Code can emit stream-json logs.
- **Transcript persistence:** The original stream-json logs were written to `/tmp/superpowers-tests/` and `/tmp/superpowers-tests/adr-pressure/`. Those paths are transient (`/tmp` is wiped on reboot/cleanup) and the original logs are no longer available. The After-change results above are the recorded observations from those runs; when opening a PR, re-run the scenarios so the transcripts can be attached to the PR description as the durable evidence trail.
