# ADR Workflow Propagation Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Turn the current "skills that say *write an ADR*" into a system where ADR state propagates as a workflow artifact through every stage of the development flow.

**Architecture:** The plan rests on three structural moves. (1) **Single source of classification** — the "does this need an ADR" decision logic lives only in `writing-architecture-decision-records`; every other skill references it instead of copying the 4 questions. (2) **The plan header is the state token** — `writing-plans` records `**ADR:**` (a path, or `Not required — <reason>`) in the plan header, and all downstream skills *read* that field instead of re-deriving the classification. (3) **Propagation into execution units** — the implementer and reviewer subagent prompts carry ADR checks, so design drift is caught where work actually happens, not only in the controller skill. The change is dogfooded: ADR-001 governs this plan itself.

**Tech Stack:** Markdown skill files; bash test harness (`tests/skill-triggering/`). No application code. Skill-content edits cannot use literal RED-GREEN-REFACTOR — the behavioral test for skills is the model eval in Task 8; per-task verification uses concrete `grep`/consistency checks.

**ADR:** docs/adr/001-adr-governed-development-flow.md

**Out of scope (Phase 2):** Mechanical enforcement via git hooks. Deferred deliberately — the core library targets multiple harnesses, so a Claude/Codex-specific hook must not be mandatory. Phase 2 should be a generic `scripts/check-adr-required` plus *optional* hook integration, planned separately once the workflow + eval are stable.

---

### Task 1: Write the governing ADR (dogfood + canonical example)

Creates `docs/adr/` and the ADR that authorizes this whole plan. By the repo's own rules this change is a workflow-responsibility/pattern change, so it requires an ADR. Writing it first locks the design and gives the skill library a real worked example.

**Files:**
- Create: `docs/adr/001-adr-governed-development-flow.md`

- [ ] **Step 1: Write the ADR**

Create `docs/adr/001-adr-governed-development-flow.md` with exactly this content:

```markdown
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
```

- [ ] **Step 2: Verify the ADR has the mechanically checkable acceptance criteria**

Run: `test -f docs/adr/001-adr-governed-development-flow.md && grep -q "Negative:" docs/adr/001-adr-governed-development-flow.md && grep -q "rejected" docs/adr/001-adr-governed-development-flow.md && echo OK`
Expected: `OK` (file exists, has a negative trade-off, has a rejected alternative).

- [ ] **Step 3: Manually verify the semantic acceptance criterion**

Read the ADR as if you had not read the implementation plan. Confirm you can answer: "Why was ADR state made a plan-header workflow artifact instead of per-skill re-classification?" without reading code or this plan. If not, revise the Situation/Complication/Answer before committing.

Expected: The ADR itself explains the reason: copied criteria drift and execution subagents do not see unpropagated ADR requirements.

- [ ] **Step 4: Commit**

```bash
git add docs/adr/001-adr-governed-development-flow.md
git commit -m "docs(adr): add ADR-001 governing the ADR-driven development flow"
```

---

### Task 2: Make `writing-architecture-decision-records` the single source of classification

Tighten the over-broad claim, give the classification table a stable anchor other skills can point to, and lock the path/numbering/status conventions so the rest of the plan has one definition to reference.

**Files:**
- Modify: `skills/writing-architecture-decision-records/SKILL.md`

- [ ] **Step 1: Narrow the absolute claim**

Replace line 10:

```
Every design decision lives in an ADR. No exceptions.
```

with:

```
Every *design-affecting* decision lives in an ADR — one that changes boundaries, responsibilities, patterns, dependencies, data flow, or corrects a design-flaw root cause. Decisions that leave the design identical do not.
```

- [ ] **Step 2: Mark the classification table as the single source of truth**

Add this paragraph directly under the `## When to Write or Update` heading, before the existing "Classify the change before deciding:" line:

```
**This table is the single source of truth for ADR classification.** Other skills MUST reference this section rather than restating the criteria.
```

- [ ] **Step 3: Add numbering rule and unify status vocabulary**

In the `## Rules` section, append these bullets:

```
- Number ADRs with zero-padded, monotonically increasing integers (`001`, `002`, ...). The next number is one above the highest in `docs/adr/`.
- Status vocabulary is exactly: `Proposed`, `Accepted`, `Superseded by ADR-NNN`. Do not use `Deprecated`.
```

In the `## Format` template, change the Status line from:

```
Proposed / Accepted / Deprecated by ADR-NNN
```

to:

```
Proposed / Accepted / Superseded by ADR-NNN
```

- [ ] **Step 4: Verify**

Run: `cd skills/writing-architecture-decision-records && grep -q "design-affecting" SKILL.md && grep -q "single source of truth for ADR classification" SKILL.md && ! grep -q "Deprecated by ADR" SKILL.md && echo OK`
Expected: `OK`

- [ ] **Step 5: Commit**

```bash
git add skills/writing-architecture-decision-records/SKILL.md
git commit -m "refactor(adr-skill): scope to design-affecting changes, anchor classification as single source"
```

---

### Task 3: Make `writing-plans` carry ADR state in the plan header

The plan header becomes the canonical state token. Replace the copied 4-question block with a reference to Task 2's anchor, and require the classification result to be written into the header.

**Files:**
- Modify: `skills/writing-plans/SKILL.md`

- [ ] **Step 1: Add the `ADR:` field to the plan header template**

In the `## Plan Document Header` template block, after the `**Tech Stack:** [Key technologies/libraries]` line, add:

```
**ADR:** [docs/adr/NNN-<short-name>.md  —OR—  Not required — <one-line reason>]
```

- [ ] **Step 2: Replace the `## ADR Reference` section**

Replace the entire current `## ADR Reference` section (the heading plus the 4 numbered questions and the two `If NO / If YES` paragraphs and the trailing "Implementation tasks must remain consistent..." paragraph) with:

```
## ADR Reference

Before defining tasks, classify the change using superpowers:writing-architecture-decision-records (see its classification table — the single source of truth). Do not restate the criteria here.

- If **not required**: set the plan header to `**ADR:** Not required — <reason>`.
- If **required**: ensure the ADR exists at `docs/adr/NNN-<short-name>.md` (write it via the ADR skill if missing), then set the plan header to `**ADR:** docs/adr/NNN-<short-name>.md`.

The header `**ADR:**` field is the workflow state token: downstream skills read it instead of re-classifying. If task definition later reveals a decision not captured in the ADR, update the ADR and the header before proceeding.
```

- [ ] **Step 3: Verify**

Run: `cd skills/writing-plans && grep -q '\*\*ADR:\*\*' SKILL.md && grep -q "single source of truth" SKILL.md && ! grep -q "Does this change modify any boundary" SKILL.md && echo OK`
Expected: `OK` (header field present, references the single source, copied questions gone).

- [ ] **Step 4: Commit**

```bash
git add skills/writing-plans/SKILL.md
git commit -m "feat(writing-plans): record ADR state in plan header, reference single-source classification"
```

---

### Task 4: Resolve the `brainstorming` contradiction

`brainstorming` tells the agent to invoke the ADR skill *and* says the only skill it may invoke next is `writing-plans`. Reframe the ADR step as a documentation gate, and replace the inline criteria with a reference.

**Files:**
- Modify: `skills/brainstorming/SKILL.md`

- [ ] **Step 1: Fix the contradictory instruction**

Replace line 143:

```
- Do NOT invoke any other skill. writing-plans is the next step.
```

with:

```
- The only implementation-*planning* skill after brainstorming is writing-plans. Writing or updating the ADR via superpowers:writing-architecture-decision-records is a documentation gate that happens *before* that transition — it is part of this step, not a detour.
```

- [ ] **Step 2: Replace the inline criteria in the Implementation section**

Replace the Implementation-section bullet:

```
- **Classify the change.** If it affects design, write the ADR using superpowers:writing-architecture-decision-records and commit it.
```

with:

```
- **Classify the change** using superpowers:writing-architecture-decision-records (its classification table is the single source of truth). If an ADR is required, write it to `docs/adr/NNN-<short-name>.md` and commit it before transitioning.
```

- [ ] **Step 3: Align the checklist item wording**

Replace checklist item 9:

```
9. **Classify and write ADR if required** — If the change affects design (boundaries, responsibilities, patterns, dependencies), document it in `docs/adr/NNN-<topic>.md` using SCQA format.
```

with:

```
9. **Classify and write ADR if required** — Classify via superpowers:writing-architecture-decision-records; if required, write it to `docs/adr/NNN-<short-name>.md` and commit. This is a documentation gate before the writing-plans transition.
```

- [ ] **Step 4: Verify**

Run: `cd skills/brainstorming && grep -q "documentation gate" SKILL.md && grep -q "NNN-<short-name>" SKILL.md && ! grep -q "Do NOT invoke any other skill" SKILL.md && ! grep -q "NNN-<topic>" SKILL.md && echo OK`
Expected: `OK`

- [ ] **Step 5: Commit**

```bash
git add skills/brainstorming/SKILL.md
git commit -m "fix(brainstorming): resolve ADR-vs-writing-plans contradiction, reference single-source classification"
```

---

### Task 5: Execution skills read the plan header instead of re-deriving

`executing-plans` and `subagent-driven-development` currently classify "before reading the plan" — but the classification material is *in* the plan. Reorder: read the plan, read its `**ADR:**` header field, act on it.

**Files:**
- Modify: `skills/executing-plans/SKILL.md`
- Modify: `skills/subagent-driven-development/SKILL.md`

- [ ] **Step 1: Rewrite `executing-plans` Step 0 + Step 1**

Replace the `### Step 0: Classify the Change` section (heading + 4 questions + the two `If YES / If NO` lines) and the `### Step 1: Load and Review Plan` numbered list with:

```
### Step 1: Load Plan and Read ADR State
1. Read plan file.
2. Read the plan header `**ADR:**` field.
   - If it names a path: verify that ADR exists and the plan is consistent with it. If the ADR is missing, stop and invoke superpowers:writing-architecture-decision-records.
   - If it says `Not required — <reason>`: sanity-check the reason against the ADR skill's classification table; if it looks misclassified, raise it with your human partner.
   - If the field is absent (older plan): classify the change yourself via superpowers:writing-architecture-decision-records, then proceed.
3. Review critically - identify any questions or concerns about the plan.
4. If concerns: Raise them with your human partner before starting.
5. If no concerns: Create TodoWrite and proceed.
```

(Renumber the following `### Step 2` / `### Step 3` headings if needed — they already read `Step 2`/`Step 3`, so no change required.)

- [ ] **Step 2: Rewrite the `subagent-driven-development` classification block**

Replace the `**Change classification:**` paragraph (4 questions + `If YES / If NO` lines) and the `**ADR check (if required):**` paragraph with:

```
**ADR state:** Read the plan, then read its header `**ADR:**` field.
- If it names a path: verify that ADR exists and the implementation will stay consistent with it. If missing, stop and invoke superpowers:writing-architecture-decision-records.
- If it says `Not required`: sanity-check against the classification table in superpowers:writing-architecture-decision-records.
- If absent (older plan): classify via superpowers:writing-architecture-decision-records before dispatching any subagent.

The controller and the implementer subagent share responsibility for keeping the ADR in sync; the implementer prompt carries the concrete check.
```

- [ ] **Step 3: Update the `subagent-driven-development` process graph**

In the `digraph process` block, replace the two ADR nodes/edges added previously:

```
    "Classify change (ADR required?)" [shape=diamond];
    "Verify ADR exists" [shape=box];
```
and
```
    "Classify change (ADR required?)" -> "Verify ADR exists" [label="yes"];
    "Classify change (ADR required?)" -> "Read plan, extract all tasks with full text, note context, create TodoWrite" [label="no"];
    "Verify ADR exists" -> "Read plan, extract all tasks with full text, note context, create TodoWrite";
```
Replace the existing read-plan node:
```
    "Read plan, extract all tasks with full text, note context, create TodoWrite" [shape=box];
```
with three sequential nodes:
```
    "Read plan file" [shape=box];
    "Read ADR header field, verify/classify" [shape=box];
    "Extract all tasks with full text, note context, create TodoWrite" [shape=box];
```

Then replace the direct read-plan → dispatch edge with:
```
    "Read plan file" -> "Read ADR header field, verify/classify";
    "Read ADR header field, verify/classify" -> "Extract all tasks with full text, note context, create TodoWrite";
    "Extract all tasks with full text, note context, create TodoWrite" -> "Dispatch implementer subagent (./implementer-prompt.md)";
```

Remove the now-dangling edge:
```
    "Read plan, extract all tasks with full text, note context, create TodoWrite" -> "Dispatch implementer subagent (./implementer-prompt.md)";
```

The ADR gate must happen before TodoWrite creation so a missing or misclassified ADR stops execution before task state exists.

- [ ] **Step 4: Verify**

Run: `grep -q "Read ADR State" skills/executing-plans/SKILL.md && ! grep -q "Step 0" skills/executing-plans/SKILL.md && ! grep -q "Before loading the plan" skills/executing-plans/SKILL.md && grep -q '\*\*ADR:\*\*' skills/subagent-driven-development/SKILL.md && ! grep -q "Before reading the plan" skills/subagent-driven-development/SKILL.md && ! grep -q "Classify change (ADR required?)" skills/subagent-driven-development/SKILL.md && grep -q "Read ADR header field" skills/subagent-driven-development/SKILL.md && echo OK`
Expected: `OK` (prose reordered, the old `Step 0` heading and the old graph nodes are gone, the new graph node is present).

- [ ] **Step 5: Commit**

```bash
git add skills/executing-plans/SKILL.md skills/subagent-driven-development/SKILL.md
git commit -m "fix(execution): read ADR state from plan header instead of re-deriving before plan load"
```

---

### Task 6: Propagate ADR responsibility into subagent and reviewer prompts

`subagent-driven-development` declares the implementer "responsible for ADR sync," but none of the three prompt templates mention ADR. Wire the check into the units that do the work.

**Files:**
- Modify: `skills/subagent-driven-development/implementer-prompt.md`
- Modify: `skills/subagent-driven-development/spec-reviewer-prompt.md`

- [ ] **Step 1: Add ADR handling to the implementer prompt**

In `implementer-prompt.md`, in the `## Your Job` numbered list, after `1. Implement exactly what the task specifies`, insert a new numbered item (renumber the rest):

```
    2. If the work introduces a design-affecting change that diverges from the plan's ADR (or the plan declared an ADR and your implementation contradicts it), STOP and report it — do not silently proceed. Update the ADR only if the controller confirms.
```

In the `## Before Reporting Back: Self-Review` section, add a new block before "If you find issues during self-review...":

```
    **ADR alignment:**
    - If the plan header named an ADR, does my implementation match it?
    - Did I introduce a design decision the ADR does not cover?
    - If either is true and I did not resolve it: report DONE_WITH_CONCERNS or NEEDS_CONTEXT.
```

- [ ] **Step 2: Add an ADR check to the spec reviewer prompt**

In `spec-reviewer-prompt.md`, in the `## Your Job` section, after the `**Misunderstandings:**` block, add:

```
    **ADR alignment (only if the plan header named an ADR):**
    - Read the ADR. Does the implementation match the decision it records?
    - Did the implementer introduce a design-affecting change the ADR does not cover?
    - Report any divergence explicitly with file:line references.
```

- [ ] **Step 3: Confirm the code quality reviewer inherits ADR alignment (no edit)**

`code-quality-reviewer-prompt.md` delegates to `requesting-code-review/code-reviewer.md` ("Use template at ..."). Task 7 Step 2 adds the ADR alignment block to that shared template, so the code quality reviewer inherits the check. Adding a second copy here would duplicate it and violate the single-source principle this plan enforces — so make **no edit**. This step is a no-op confirmation.

Run: `grep -q "code-reviewer.md" skills/subagent-driven-development/code-quality-reviewer-prompt.md && echo OK`
Expected: `OK` (the prompt still delegates to the shared template that Task 7 Step 2 updates).

- [ ] **Step 4: Verify**

Run: `cd skills/subagent-driven-development && grep -q -i adr implementer-prompt.md && grep -q -i adr spec-reviewer-prompt.md && echo OK`
Expected: `OK` (the two standalone prompt files now reference ADR; the code quality reviewer inherits it via `code-reviewer.md` — see Task 7 Step 2).

- [ ] **Step 5: Commit**

```bash
git add skills/subagent-driven-development/implementer-prompt.md skills/subagent-driven-development/spec-reviewer-prompt.md
git commit -m "feat(sdd-prompts): propagate ADR sync responsibility into implementer and spec-reviewer prompts"
```

---

### Task 7: Reference-not-copy sweep + convention unification across remaining skills

`requesting-code-review`, `receiving-code-review`, `verification-before-completion`, and `finishing-a-development-branch` should read the plan's `**ADR:**` field and reference the single-source classification. Also unify the path placeholder and status vocabulary everywhere.

**Files:**
- Modify: `skills/requesting-code-review/SKILL.md`
- Modify: `skills/requesting-code-review/code-reviewer.md`
- Modify: `skills/receiving-code-review/SKILL.md`
- Modify: `skills/verification-before-completion/SKILL.md`
- Modify: `skills/finishing-a-development-branch/SKILL.md`

- [ ] **Step 1: `requesting-code-review` — reference the header field**

In the `**2. Verify ADR alignment...**` step, replace the phrase "If this task involved changes to boundaries, responsibilities, patterns, or dependencies" with "If the plan header named an ADR (see superpowers:writing-architecture-decision-records for classification)". Keep the rest of the step.

- [ ] **Step 2: `requesting-code-review/code-reviewer.md` — make ADR drift part of standalone review**

In `skills/requesting-code-review/code-reviewer.md`, after the `**Head:** {HEAD_SHA}` line, add:

```
    **ADR:** {ADR_STATUS_OR_PATH}
```

In the `## What to Check` section, after the `**Plan alignment:**` block, add:

```
    **ADR alignment:**
    - If ADR names a path: read it and verify the implementation still matches the recorded decision.
    - Did the implementation introduce a design-affecting decision not covered by the ADR?
    - Report code/ADR drift at Important severity or higher.
```

In `skills/requesting-code-review/SKILL.md`, add `{ADR_STATUS_OR_PATH}` to the placeholder list with:

```
- `{ADR_STATUS_OR_PATH}` — ADR path from the plan header, `Not required — <reason>`, or `Unknown` if no plan/header exists
```

- [ ] **Step 3: `verification-before-completion` — reference classification, not inline criteria**

In `CHECK ADR` step 5, replace the parenthetical "(only if changes affect boundaries, responsibilities, patterns, or dependencies)" with "(only if the plan header named an ADR, or — absent a plan — superpowers:writing-architecture-decision-records classifies the change as design-affecting)".

- [ ] **Step 4: `finishing-a-development-branch` — reference the header field in the ADR Gate**

In the `### ADR Gate` section, replace the line:

```
**If design-affecting changes were made:**
- Check `docs/adr/` for the relevant ADR.
```

with:

```
**If the plan header named an ADR (or, absent a plan, superpowers:writing-architecture-decision-records classifies the change as design-affecting):**
- Check `docs/adr/` for that ADR and confirm it matches the implementation.
```

- [ ] **Step 5: `receiving-code-review` — no criteria copied, leave as-is but verify wording**

`receiving-code-review` only references "the ADR" without copying criteria — no edit needed. This step is a no-op confirmation: `grep -c "Does this change modify any boundary" skills/receiving-code-review/SKILL.md` must return `0`.

- [ ] **Step 6: Safety sweep — confirm path placeholder and status vocabulary are unified**

Tasks 2–5 already convert every `docs/adr/NNN-<feature-name>.md` / `docs/adr/NNN-<topic>.md` to `docs/adr/NNN-<short-name>.md` and every `Deprecated by ADR` to `Superseded by ADR`, and the Task 7 files (`requesting-code-review`, `receiving-code-review`, `verification-before-completion`, `finishing-a-development-branch`) never used the `NNN-` placeholders. This step is a safety net, not new bulk work.

Run: `grep -rn 'NNN-<feature-name>\|NNN-<topic>\|Deprecated by ADR' skills/ || echo "none — already clean"`
If anything is listed, fix those files by hand (do not bulk-replace — behavior-shaping prose must not change unreviewed), then run `git diff -- skills/` and inspect before proceeding.

- [ ] **Step 7: Verify**

Run: `! grep -rq 'NNN-<feature-name>\|NNN-<topic>\|Deprecated by ADR' skills/ && [ "$(grep -rl 'Does this change modify any boundary' skills/ | wc -l)" = "0" ] && grep -q "ADR alignment" skills/requesting-code-review/code-reviewer.md && grep -q "{ADR_STATUS_OR_PATH}" skills/requesting-code-review/SKILL.md && echo OK`
Expected: `OK` (no stale placeholders/status terms; copied criteria gone outside the ADR skill; standalone code reviewer now checks ADR drift).

- [ ] **Step 8: Commit**

```bash
git add skills/requesting-code-review/SKILL.md skills/requesting-code-review/code-reviewer.md skills/receiving-code-review/SKILL.md skills/verification-before-completion/SKILL.md skills/finishing-a-development-branch/SKILL.md skills/writing-plans/SKILL.md skills/brainstorming/SKILL.md skills/executing-plans/SKILL.md skills/subagent-driven-development/SKILL.md
git commit -m "refactor(skills): reference single-source ADR classification, unify path and status conventions"
```

---

### Task 8: Add the skill-triggering eval and document pressure tests

Per `CLAUDE.md` ("Skill Changes Require Evaluation"), the new skill needs a triggering test in the harness plus documented adversarial scenarios.

**Files:**
- Create: `tests/skill-triggering/prompts/writing-architecture-decision-records.txt`
- Modify: `tests/skill-triggering/run-all.sh`
- Create: `tests/skill-triggering/ADR-PRESSURE-TESTS.md`
- Create: `docs/superpowers/evals/2026-05-14-adr-workflow-propagation.md`

- [ ] **Step 1: Create the triggering prompt**

Create `tests/skill-triggering/prompts/writing-architecture-decision-records.txt` with:

```
We've decided to stop calling the payments provider synchronously from the
checkout handler. Instead we'll publish an event and have a separate worker
settle the charge asynchronously. This moves the settlement responsibility out
of the request path and changes how the order service and the payments worker
talk to each other. Let's get this decision written down before we start.
```

(A naive prompt describing a boundary/responsibility change — should trigger `writing-architecture-decision-records`.)

- [ ] **Step 2: Register the skill in `run-all.sh`**

In `tests/skill-triggering/run-all.sh`, in the `SKILLS=(` array, after the `"requesting-code-review"` line, add:

```
    "writing-architecture-decision-records"
```

- [ ] **Step 3: Document the adversarial pressure-test scenarios**

Create `tests/skill-triggering/ADR-PRESSURE-TESTS.md` with:

```markdown
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
```

- [ ] **Step 4: Create the evaluation results document**

Create `docs/superpowers/evals/2026-05-14-adr-workflow-propagation.md` with:

```markdown
# ADR Workflow Propagation Eval Results

## Environment

| Harness | Harness version | Model | Model version/ID | Date |
|---------|-----------------|-------|------------------|------|
| Claude Code | <fill from run> | <fill from run> | <fill from run> | 2026-05-14 |

## Scenario Results

| # | Scenario | Baseline result | After-change result | Pass? | Transcript/log |
|---|----------|-----------------|---------------------|-------|----------------|
| 1 | New component / new boundary added | <summary> | <summary> | <yes/no> | <path or pasted excerpt> |
| 2 | Typo fix / null check / rename only | <summary> | <summary> | <yes/no> | <path or pasted excerpt> |
| 3 | Plan header names missing ADR file | <summary> | <summary> | <yes/no> | <path or pasted excerpt> |
| 4 | Implementation diverges from named ADR | <summary> | <summary> | <yes/no> | <path or pasted excerpt> |

## Notes

- Scenario 1 is covered by `tests/skill-triggering/run-test.sh`.
- Scenarios 2-4 are manual pressure tests and must be filled in before PR/merge.
- If any scenario fails, tune the relevant skill/prompt and re-run that scenario before completing the branch.
```

- [ ] **Step 5: Verify static eval assets**

Run: `test -f tests/skill-triggering/prompts/writing-architecture-decision-records.txt && grep -q "writing-architecture-decision-records" tests/skill-triggering/run-all.sh && test -f tests/skill-triggering/ADR-PRESSURE-TESTS.md && test -f docs/superpowers/evals/2026-05-14-adr-workflow-propagation.md && bash -n tests/skill-triggering/run-all.sh && echo OK`
Expected: `OK` (prompt exists, skill registered, pressure-test doc exists, eval results doc exists, `run-all.sh` still parses).

- [ ] **Step 6: Run the triggering eval (manual — requires `claude` CLI + API)**

Run: `cd tests/skill-triggering && ./run-test.sh writing-architecture-decision-records prompts/writing-architecture-decision-records.txt 3`
Expected: `✅ PASS: Skill 'writing-architecture-decision-records' was triggered`. If it fails, tune the skill `description:` and/or the prompt and re-run; record before/after in the PR.

- [ ] **Step 7: Run and record the manual pressure tests**

Run scenarios 2-4 from `tests/skill-triggering/ADR-PRESSURE-TESTS.md` manually. Fill in every `<summary>`, `<yes/no>`, and transcript/log field in `docs/superpowers/evals/2026-05-14-adr-workflow-propagation.md`.

Expected:
- Scenario 2: no ADR is written; the agent classifies the change as not required with a concrete reason.
- Scenario 3: `executing-plans` / `subagent-driven-development` stop before TodoWrite/task execution when the named ADR file is missing.
- Scenario 4: spec reviewer and code-quality reviewer flag code/ADR drift; direct `requesting-code-review` also flags drift via `code-reviewer.md`.

- [ ] **Step 8: Verify eval results are filled in**

Run: `! grep -q '<summary>\|<yes/no>\|<path or pasted excerpt>\|<fill from run>' docs/superpowers/evals/2026-05-14-adr-workflow-propagation.md && grep -q "diverges from named ADR" docs/superpowers/evals/2026-05-14-adr-workflow-propagation.md && echo OK`
Expected: `OK` (the eval document no longer contains placeholders and still includes the ADR drift scenario row).

- [ ] **Step 9: Commit**

```bash
git add tests/skill-triggering/prompts/writing-architecture-decision-records.txt tests/skill-triggering/run-all.sh tests/skill-triggering/ADR-PRESSURE-TESTS.md docs/superpowers/evals/2026-05-14-adr-workflow-propagation.md
git commit -m "test(skill-triggering): add ADR skill triggering eval and pressure-test scenarios"
```

---

### Task 9: Update the README

Add the ADR skill to the public-facing workflow description and skills list.

**Files:**
- Modify: `README.md`

- [ ] **Step 1: Add the ADR gate to "The Basic Workflow"**

In `README.md`, in the numbered "The Basic Workflow" list, insert a new item between item 1 (`brainstorming`) and item 2 (`using-git-worktrees`), and renumber the rest:

```
2. **writing-architecture-decision-records** - Activates when a design-affecting decision is made. Records the decision (situation, trade-offs, rejected alternatives) in `docs/adr/`. Acts as a documentation gate before planning begins.
```

- [ ] **Step 2: Add the skill to "What's Inside"**

In the `### Skills Library` section, under `**Collaboration**`, add:

```
- **writing-architecture-decision-records** - One-page ADRs for design-affecting decisions
```

- [ ] **Step 3: Verify**

Run: `grep -c "writing-architecture-decision-records" README.md`
Expected: `2` (or more) — appears in both the workflow list and the skills list.

- [ ] **Step 4: Commit**

```bash
git add README.md
git commit -m "docs(readme): document the writing-architecture-decision-records skill"
```

---

## Self-Review

**1. Spec coverage** — every consolidated finding maps to a task:
- Codex#1 (ADR has no ADR) → Task 1.
- Codex#2 / brainstorming contradiction → Task 4.
- Codex#3 / classify-before-reading-plan → Task 5; plan-header state token → Task 3.
- Codex#4 / subagent prompt propagation → Task 6.
- Codex#5 / missing eval → Task 8.
- Codex follow-up / standalone code reviewer missing ADR drift checks → Task 7.
- DRY duplication of the 4 questions → Tasks 2, 3, 4, 5, 7.
- README → Task 9. Path/status convention drift → Tasks 2, 7.
- "Every design decision" too strong → Task 2.
- Hook enforcement → explicitly deferred to Phase 2 (Out of scope).

**2. Placeholder scan** — no "TBD"/"handle edge cases"/"similar to Task N". Every edit step shows exact old/new text or exact insert text; every verification step is a runnable command with expected output.

**3. Type consistency** — the load-bearing identifiers are consistent across tasks: the header field is `**ADR:**` everywhere (Tasks 3, 5, 6, 7); the path convention is `docs/adr/NNN-<short-name>.md` everywhere (Tasks 2, 4, 7); the status set is `Proposed | Accepted | Superseded by ADR-NNN` (Tasks 1, 2, 7); the classification table is marked "single source of truth for ADR classification" in the ADR skill (Task 2) and every other skill references it by that phrase rather than restating the criteria (Tasks 3, 4, 5, 7).

## Execution Handoff

Task ordering is dependency-correct: Task 1 locks the design, Task 2 establishes the single source the rest reference, Task 3 defines the state token, Tasks 4–7 wire producers and consumers, Task 8 adds the eval, Task 9 documents. Tasks 4 and 5 may run in parallel after Task 3; Task 6 is independent of Tasks 4–5 once Task 3 is done.
