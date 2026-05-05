---
name: writing-architecture-decision-records
description: Use when a design decision is made, modified, or added that affects boundaries, responsibilities, patterns, or dependencies.
---

# Writing Architecture Decision Records

## Overview

Every design decision lives in an ADR. No exceptions.

**Core principle:** If the design changed, the ADR changed.

## When to Write or Update

Classify the change before deciding:

| Change Type | ADR Required? | Examples |
|---|---|---|
| **New feature, component, pattern** | ✅ Yes | New API endpoint, new module, new framework |
| **Modified boundary, interface, data flow** | ✅ Yes | Contract changes, event flow changes, schema changes |
| **New or replaced dependency** | ✅ Yes | New library, version upgrade with breaking changes |
| **Override or deprecation of prior decision** | ✅ Yes | Replacing pattern, retiring component |
| **Refactoring with responsibility shift** | ✅ Yes | Moving logic between layers, redefining boundaries |
| **Performance fix (algorithm/strategy change)** | ✅ Yes | Caching strategy, async conversion, complexity reduction |
| **Design-flaw root-cause fix** | ✅ Yes | Fix that corrects a prior architectural mistake |
| **Simple bug fix (logic, typo, null check)** | ❌ No | Off-by-one, missing validation, typo |
| **Naming or formatting change** | ❌ No | Rename variable, add type annotation, linter fix |
| **Simple extraction (no responsibility shift)** | ❌ No | Extract function within same file, inline variable |

**Rule of thumb:** If the change alters who is responsible for what, how components talk to each other, or what patterns are used → ADR required. If the change keeps the design identical and only fixes or cleans up → ADR not required.

**Multi-step refactoring:** If a refactor is planned in stages (e.g., extract then move), write the ADR when the first design-affecting step begins. Don't wait until the final move.

**Uncertain? Default to ADR.**
If you're unsure whether a change requires an ADR, write one. A one-page ADR for a simple change is cheap. A missing ADR for a design change is expensive.

## When to Update (Not Just Write)

- Before implementation begins
- When review reveals the ADR was wrong or incomplete
- When implementation diverges from the ADR — fix the ADR or the code, never leave them out of sync

## Format

One page maximum. File: `docs/adr/NNN-<short-name>.md`

```markdown
# ADR-NNN: Title

## Situation
What context led to this decision? (2-3 sentences)

## Complication
What constraint, risk, or trade-off forced a choice? (2-3 sentences)

## Question
What exactly are we deciding? (1 sentence)

## Answer
What did we choose and why?

- Decision
- Consequences (positive and negative)
- Alternatives considered and rejected (1 sentence each)

## Status
Proposed / Accepted / Deprecated by ADR-NNN
```

## Acceptance Criteria

An ADR is complete only when:
- [ ] A reader can understand WHY the decision was made without reading the code
- [ ] Consequences include at least one negative trade-off (no decision is free)
- [ ] Alternatives section has at least one rejected option with reason

Incomplete ADRs get sent back. "Decision: Use X. Consequences: Good." is not acceptable.

## Rules

- One decision per ADR
- Link to related ADRs: `Supersedes ADR-012`, `Depends on ADR-005`
- Keep it under one page — if it needs more, the decision isn't crisp enough

## Red Flags - STOP

- Starting implementation without an ADR
- ADR and code out of sync
- ADR longer than one page
- "We'll write it later"
- Copy-pasting the spec instead of stating the decision
- Commits that change ADR-related code without referencing the ADR
