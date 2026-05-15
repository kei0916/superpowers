# ADR-001: ADR-governed development flow

## Situation
The superpowers skill library drives a multi-stage development workflow (brainstorm → plan → execute → review → finish). Design decisions made during this flow were not durably recorded, so reviewers and future implementers had to reconstruct intent from code.

## Complication
Simply adding a "write an ADR" skill is not enough: unless ADR state is carried between stages, each skill re-derives whether an ADR is needed, the criteria drift across copies, and execution subagents never see the requirement at all.

## Question
How should the ADR requirement be represented and propagated so that every stage of the workflow enforces it consistently?

## Answer
Adopt ADR-governed flow with three structural rules:

- **Decision:**
  1. Classification logic ("does this change need an ADR") lives only in `writing-architecture-decision-records`. Other skills reference it; they do not copy the criteria.
  2. `writing-plans` records the result in the plan header as `**ADR:** <path>` or `**ADR:** Not required — <reason>`. This header field is the canonical workflow state token.
  3. Downstream skills (executing-plans, subagent-driven-development, code review, verification, finishing-a-development-branch) and the implementer/reviewer subagent prompts *read* that field instead of re-classifying.
  4. ADR scope is "design-affecting decisions," not literally every decision. Files are `docs/adr/NNN-<short-name>.md` with zero-padded monotonic numbers. Status vocabulary is `Proposed | Accepted | Superseded by ADR-NNN`.
- **Consequences:**
  - Positive: one place to tune criteria; execution units cannot silently skip the gate; reviewers have an explicit artifact to check against.
  - Negative: the plan header schema is now load-bearing — a plan authored by an older `writing-plans` version, or with a missing `ADR:` field, forces consumers into a fallback re-classification path. This coupling is accepted.
- **Alternatives considered and rejected:**
  - Copy the classification criteria into each skill — rejected: guaranteed drift across 8 files.
  - Enforce via a git hook — rejected for the core library: harness-specific and would not be portable; deferred to a Phase 2 optional integration.
  - Leave classification implicit per stage — rejected: that is the current state and it produced false negatives and un-propagated requirements.

## Status
Accepted
