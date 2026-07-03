# Change Budget

## Judgment Rules

Minimal scope is determined by contract closure. When a single file suffices, modify only that file; when necessary chain links such as parser → state boundary → renderer → tests must synchronize, they all enter the budget together—otherwise fewer files simply means an incomplete change.

| Decision Point | Default Action | Condition to Expand |
| --- | --- | --- |
| Single file forms complete closure | Modify only the current file | Callers, dependencies, or contract boundaries must change in sync |
| Simple condition suffices | Modify the existing check | The same bug has already been caused by duplicate logic |
| Old scheme has been rejected | Delete the old-scheme closure | Latest contract explicitly requires compatibility or rejection of that old input |
| Shared code still serves current behavior | Retain and avoid rewriting | Current contract requires synchronous boundary adjustment |
| New helper needed | Inline local logic first | 3+ real reuse sites or test-boundary requirement |
| Type/enum change needed | Modify only items reached by the current requirement | API contract or compilation error requires synchronization |

## Budget Writing Requirements

Before any edits, list actual exact paths; enumerate every path touched by the superseded scheme; explicit "no change" items prove that shared behavior and non-goals are protected.

```markdown
## Change Budget (pre-implementation)
- `exact/path`: modify/delete/retain - necessary reason
- No change: `exact/path` - reason
```

The budget must cover old-scheme reachable production code, tests, fixtures, configs, comments, and copy.

## Detailed Steps

### 1. Lock the Latest Contract

Decompose the user's latest requirement into three categories:

- **Latest required behavior**: What the final code in this round must support.
- **Superseded behavior/identifiers**: Which prior-scheme helpers, old configs, old tests, old comments are no longer valid.
- **Non-goals/compatibility requirements**: Which existing behaviors must be preserved, and which should not be incidentally handled.

Why: In continuous conversation, old context creates the illusion of "supporting both A/B paths" or "keeping old tests is safer"; explicitly writing out superseded content prevents old-scheme sedimentation.

### 2. Write Change Budget Before Any Edits

Before any source or test editing, write an exact-path budget. The budget constrains the diff before editing begins, blocking scope creep during execution and showing the reviewer why each file must appear.

The budget is an execution constraint, not a gate awaiting user re-confirmation.

### 3. Implement per Minimal Complete Closure

Prefer modifying existing checks and local logic; expand to multiple files only when the current contract naturally spans entry point, state boundary, rendering, or tests. Before adding a new helper, config, compatibility switch, or abstraction, first assess whether real reuse or contract requirement exists.

Why: "Fewer files" does not equal complete, and "more elegant abstraction" does not equal necessary; the goal is the current contract's minimal runnable closure.

### 4. Clean Superseded-Scheme Residue

Starting from known old fields, old functions, old config keys, old enum values, and old copy, search for references; classify each hit as `delete`, `retain`, or `restore`. `Retain` applies only to shared implementations still depended on by the latest behavior, or compatibility boundaries explicitly requested by the user.

### 5. Control Comments

Retain only comments that explain business constraints, compatibility reasons, boundary conditions, or external contracts. Remove comments that translate code behavior. See [`cleanup.md`](./cleanup.md) for details.

### 6. Close Out in Auditable Format

The final reply must name paths, old-scheme cleanup, key boundaries, and fresh verification evidence. See [`verification.md`](./verification.md) for details.
