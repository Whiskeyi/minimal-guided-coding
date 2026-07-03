---
name: minimal-guided-coding
description: "Converges source code changes during guided / vibe coding sessions: enforces minimal-complete edits, thoroughly removes superseded-scheme residue, and produces reviewer-friendly CRs. MUST be loaded when editing source code and any of these signals appear in conversation: minimal change, too much diff, don't over-modify, converge changes, switch from scheme A to B, earlier changes were wrong, revert changes, legacy code residue, clean up residue, CR is unclear, too many comments. Even without explicit keywords, proactively load this Skill when the conversation exhibits diff bloat, superseded-scheme sedimentation, or declining review readability. NOT applicable: greenfield features, large-scale refactors, new module scaffolding, or routine incremental development."
---

# Minimal Guided Coding

## Overview

Helps the agent converge existing code changes during continuous guided coding: serve only the user's latest requirement, remove residue from previously rejected schemes, avoid introducing large abstractions, dual-path compatibility, or noise comments for a "small change", and produce a CR description with verification evidence that a reviewer can audit.

Why: The primary risk of guided programming is not "inability to write code" but the accumulation of superseded schemes in context—the agent easily carries rejected approaches, temporary compatibility shims, and opportunistic refactors into the final diff.

## Scope

When the boundary is uncertain, default to the normal development workflow; switch to this Skill when diff bloat, superseded-scheme residue, or user-requested convergence appears.

## Core Judgments

1. **Latest requirement overrides old context**: When the user rejects an old scheme, that scheme becomes cleanup-candidate residue, not a compatibility path.
2. **Minimal complete contract closure**: Not the fewest files, but the closed loop of entry point, state, rendering, and tests required by the current contract.
3. **Residue cleanup extends to tests/fixtures/comments/copy**: Changing only production code is insufficient; scattered references must also be closed out.
4. **CR readability first**: The final diff lets a reviewer quickly see why it changed, what changed, and that there is no hidden sedimentation.

## Workflow

1. **Lock contract → Write budget**: Extract the latest required behavior, superseded identifiers, and non-goals; read [`budget.md`](./references/budget.md), complete the residue search; write an exact-path budget before any edits.
2. **Implement → Clean up**: Edit code per budget; read [`cleanup.md`](./references/cleanup.md), synchronously clean imports, tests, fixtures, comments, and copy per the implementation-write rules to avoid leaking user instructions into project files. Do not retain superseded old values in final tests unless the latest contract explicitly specifies expected behavior for that old value.
3. **Audit → Verify**: Remove unrelated diff; run fresh verification commands matching the change scope, output a concise reviewable reply; for complex CRs read [`verification.md`](./references/verification.md).

The budget and verification plan are internal execution constraints, not gates awaiting user re-confirmation.

## Lightweight Execution Contract

Treat the contract as a convergence reminder, not a long template.

- **Before edits**: Name the exact paths and reasons in one sentence or a short list; a single-sentence suffices for single-file small changes, expand to a full budget for multi-file or residue-cleanup scenarios.
- **During implementation**: Only process code, tests, fixtures, comments, and copy directly reached by the latest contract; do not retain rejected old paths "just to be safe".
- **Final reply**: After a fresh run, briefly state which paths changed, which old identifiers were cleaned, and what verification was performed. Use the full structure from [`verification.md`](./references/verification.md) only for complex CRs or when a reviewer needs to cross-check.

## Safety Boundaries

- Do not install dependencies, modify lockfiles, migrate directories, run global formatters, or perform bulk reordering for a small change.—These operations exceed the current change contract, bloating the diff and making the CR unreadable.
- Do not touch remote deployments, APIs, permissions, configs, or data changes unrelated to the current requirement.—Unauthorized cross-domain side effects are extremely difficult to roll back once shipped.
- Do not include "unify style", "opportunistic cleanup", or "prepare for extensibility" in this CR.—Mixing improvement-type changes prevents the reviewer from distinguishing business changes from cosmetic changes, doubling review cost.
- When verification scope needs expansion, first prove necessity; otherwise return to the minimal closure.—Verification divergence induces new changes, breaking the locked change budget.

## Reference Routing

| When to read | File | Contents |
| --- | --- | --- |
| Writing budget, judging scope | [`budget.md`](./references/budget.md) | Change budget judgment rules, detailed steps |
| Cleaning residue, controlling writes and comments | [`cleanup.md`](./references/cleanup.md) | Scheme-switch cleanup flow, implementation-write rules, comment rules |
| Verifying and closing out | [`verification.md`](./references/verification.md) | CR clarity check, verification checklist, final reply structure |
| Uncertain boundaries, red flags | [`examples.md`](./references/examples.md) | Examples, common mistakes, risk signals |
