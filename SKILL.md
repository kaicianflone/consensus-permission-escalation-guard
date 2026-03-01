---
name: consensus-permission-escalation-guard
description: Pre-execution governance for IAM and permission escalation changes. Use when an agent or workflow proposes granting, expanding, or assuming higher privileges and you need deterministic ALLOW/BLOCK/REQUIRE_REWRITE decisions with strict schema validation, idempotency, and board-native audit artifacts.
---

# consensus-permission-escalation-guard

`consensus-permission-escalation-guard` is the final safety gate before privilege elevation is applied.

## What this skill does

- validates escalation requests against a strict input schema (reject unknown fields)
- evaluates hard-block and rewrite policy flags for IAM risk patterns
- runs persona-weighted voting (or aggregates external votes)
- returns one of: `ALLOW | BLOCK | REQUIRE_REWRITE`
- writes decision and persona update artifacts for replay/audit

## Decision policy shape

Hard-block examples:
- wildcard permissions (`*`, `: *`, broad owner/admin jumps)
- missing ticket reference when required
- break-glass escalation without incident reference
- separation-of-duties conflicts (e.g., create + approve authority)

Rewrite examples:
- weak or non-actionable justification
- temporary duration exceeds policy limit
- production escalation requires explicit human confirmation gate

## Runtime and safety model

- runtime binaries: `node`, `tsx`
- credentials: none required for local guard evaluation
- network behavior: none in deterministic guard logic (persona generation backend may be external depending on your deployment)
- filesystem writes: consensus board/state artifacts under configured state path

## Invoke contract

- `invoke(input, opts?) -> Promise<OutputJson | ErrorJson>`

Modes:
- `mode="persona"` (default): generate/load persona_set and vote internally
- `mode="external_agent"`: consume `external_votes[]`, then aggregate and enforce policy deterministically

## Quick start

```bash
cd repos/consensus-permission-escalation-guard
node --import tsx run.js --input ./examples/input.json
```

## Tests

```bash
npm test
```

Test coverage includes schema rejection, hard-block paths, rewrite paths, allow paths, idempotent retries, and external-agent aggregation behavior.
