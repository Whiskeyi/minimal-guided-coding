# minimal-guided-coding

Language: [English](./README.md) | [简体中文](./README.zh-CN.md)

## Pain Points

In vibe coding / guided programming, incremental adjustments easily balloon from "small change" to "large change":

- A local requirement triggers extra helpers, configs, abstractions, or cross-file refactors.
- The user already rejected scheme A in favor of scheme B/C, yet old code, old tests, fixtures, comments, or copy remain.
- Both old and new logic paths are kept "just to be safe," causing the final code to deviate from the latest requirement.
- The CR description says only "done," leaving the reviewer unable to tell why these files changed, whether old-scheme residue is fully cleaned, or whether verification is reproducible.
- Obvious logic gets patronizing "translation-style" comments that add noise rather than clarity.
- To explain modification intent, user instructions, task context, or "modified as requested" are unnecessarily written into code, comments, test names, or user-facing copy.
- When no test was requested and similar code in the repo has none, temporary validation tests are left in the final CR.

`minimal-guided-coding` solves this class of convergence problems: ensuring the final code serves only the user's latest requirement, completing the change in a minimal-complete closure, while cleaning up residue from rejected or outdated schemes.

## Positioning

This Skill is designed for guided modification of existing code.

"Guided" here means: the user has explicitly specified how to adjust an existing implementation across continuous conversation turns. The agent's task is to converge per the latest requirement—not to independently design large incremental features, refactor architecture, or scaffold new modules from scratch.

## When to Use

Appropriate:

- The current task is a focused adjustment to existing code.
- The user has expressed convergence intent, e.g., "don't over-modify," "too much diff," "the previous scheme was wrong," "revert old changes," "clean up residue," "CR should be clear."
- Diff scope bloat, superseded-scheme sedimentation, comment noise, or an unauditable final description has been observed or is strongly suspected.

Not appropriate:

- Foreseeable large features, large refactors, or new module scaffolding.
- Routine incremental development where the user has not expressed intent to control change scope or clean up old schemes.
- A single small fix with clear boundaries, no old-scheme residue, and no diff divergence risk.
- Scenarios requiring product exploration, scheme divergence, or agent-initiated creative expansion.

## How It Works

This Skill decomposes "minimal implementation" into 4 lightweight constraints.

| Constraint | Purpose |
| --- | --- |
| Latest contract first | The user's latest requirement overrides old context; rejected old schemes default to cleanup candidates. |
| Pre-implementation mini-budget | Before editing code, name exact paths, actions, and reasons in one sentence or short list—preventing scope creep during execution. |
| Residue closed-loop cleanup | Trace imports, helpers, types, configs, fixtures, tests, comments, and copy to check whether old-scheme identifiers persist. |
| Auditable CR | The final reply states modification scope, old-scheme cleanup, key boundaries, and fresh verification results. |

The budget and verification plan are the agent's execution constraints, not gates awaiting user re-confirmation; when the user's latest requirement is sufficiently clear, proceed to implementation immediately after writing the budget.

For multi-file or complex residue scenarios, the following budget format may be used:

```markdown
## Change Budget (pre-implementation)
- `exact/path`: modify/delete/retain - necessary reason
- No change: `exact/path` - reason
```

The final reply must cover:

- Modification scope: why each path had to change.
- Old-scheme cleanup: what was deleted, or confirmation that no old-scheme residue remains.
- Key boundaries: named grep-able fields, states, enums, thresholds, APIs, or config keys.
- Verification: actual command, exit code, total test count, pass count, and 0 failures from a fresh final run.

## Minimal Scope Principle

"Minimal" does not mean the fewest files—it means the minimal complete closure of the current contract.

When a single file achieves closure, modify only that file; when a contract naturally spans parser, state boundary, renderer, and tests, those necessary files should all enter the budget together. Conversely, generic helpers, config layers, compatibility branches, and old tests that do not belong to the current closure should not be retained "in case they might be useful later."

## Comment Rules

Comments are budget items, not courtesy items.

Only retain or add comments when:

- A business rule, compatibility constraint, or boundary condition is not obvious from code structure.
- Logic is complex enough that removing the comment would prevent a reviewer from understanding why it was written this way.
- A type, enum, or public constant carries business semantics requiring explanation of value meaning or external contract.

"Translation-style" comments on obvious branches, simple assignments, straightforward functions, or local variables must be removed.

## Evaluation Design

Evaluation cases are designed around 4 pressure scenarios, each corresponding to a real-world loss-of-control risk.

| Case | Pressure Scenario | Primary Checkpoints |
| --- | --- | --- |
| multi-turn-latest-intent | Schemes A and B are both rejected across conversation turns; only scheme C is retained. | Is only the latest access rule implemented? Are A/B helpers, configs, old enums, and old tests deleted? Is no compatibility fallback retained? |
| hidden-scattered-residue | The main behavior is correct, but the rejected scheme is scattered across policy, configs, types, fixtures, tests, and copy. | Is manual-review residue cleaned? Is audit still used by current behavior retained? Is the main flow left undisturbed? |
| tempting-shared-abstraction | The previous version extracted a generic engine, config, and explanatory comments for a small rule; review asked to narrow scope, and it's tempting to write the review request into comments or test names. | Is per-list/detail local judgment restored? Is the generic rule layer and redundant comments removed? Is user-instruction meta-description avoided? Are special encodings limited to business-motivated modification reasons? |
| necessary-cross-file-closure | A state value changes from `waiting` to `queued`, but the real contract spans parser, renderer, export state, and tests. | Is the necessary cross-file closure complete? Are `running` and `done` original behaviors preserved? Is old `waiting` residue removed? |

Each case runs both with-skill and baseline configurations.

| Configuration | Guidance Approach |
| --- | --- |
| baseline | Uses the same fixture and business prompt, but only requests standard coding completion; does not provide the `minimal-guided-coding` Skill path, pre-implementation budget, key-boundary template, or residue-closure workflow. |
| with-skill | On top of the same business prompt, explicitly requires reading the Skill first and executing per the Skill's budget, cleanup, and final CR contract. |

This design ensures that differences primarily stem from whether the Skill intervenes, not from different business requirements.

Automated assertions check not only whether `node --test` passes, but also:

- Whether behavior satisfies the latest contract.
- Whether old-scheme identifiers, files, tests, fixtures, or copy still have residue.
- Whether code, comments, fixtures, test names, or user-facing copy contain user-instruction meta-description; whether special encoding motivations are business-phrased and brief.
- Whether modified files fall exclusively within the necessary closure.
- Whether tests actually exercise key boundaries rather than merely containing keywords.
- Whether a pre-edit change budget exists.
- Whether the final CR names key boundaries and provides fresh `node --test` evidence.
- Whether code metrics use LCS line-level diff counting to avoid underestimating repeated lines.

## Evaluation Results

Current complex evaluation uses the final-curated caliber, from 4 complex cases with valid fresh agent runs; code metrics use LCS line-level changed-line count.

| Metric | with-skill final | baseline realistic | Delta |
| --- | ---: | ---: | ---: |
| Assertion pass rate | 100% (30/30) | 63.3% (19/30) | +36.7% |
| Changed files | 19 | 20 | -1 |
| Lines added | +14 | +25 | -11 |
| Lines deleted | -121 | -118 | +3 |

Per-case results:

| Case | with-skill final | baseline realistic | Primary Difference |
| --- | ---: | ---: | --- |
| multi-turn-latest-intent | 7/7 | 3/7 | Baseline retains A/B old identifiers, lacks budget and CR evidence. |
| hidden-scattered-residue | 8/8 | 6/8 | Baseline cleans main behavior but lacks pre-edit budget and reproducible CR evidence. |
| tempting-shared-abstraction | 8/8 | 6/8 | Baseline achieves code narrowing but lacks budget and precise verification report. |
| necessary-cross-file-closure | 7/7 | 4/7 | Baseline retains `waiting` residue, CR boundary description incomplete. |

The `100%` here refers to the final curated re-evaluation result under 4 complex cases with 30 automated assertions, and does not represent that all real-world scenarios will achieve 100% in a single pass.

This result demonstrates: the Skill's primary benefit is not mechanically reducing file count, but eliminating superseded-scheme residue, reducing unnecessary additions, and making the final CR and verification evidence auditable.
