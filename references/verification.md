# Verification and CR Clarity

## CR Clarity Check

Check each item before submission:

- Every modified file can be justified in one sentence.
- The diff contains no unrelated formatting, bulk sorting, opportunistic renaming, file moves, or style unification.
- Cross-file changes have an explicit causal chain: entry-point change → contract change → callers/tests synchronized.
- New logic is no more general than the requirement demands; no premature support for unrequested patterns.
- Comment count and placement pass the [`cleanup.md`](./cleanup.md) comment rules check.
- Dead imports, unused types, old tests, old copy, and dead branches have been removed.

## Verification Checklist

- Confirm against Scope that the task still qualifies as "incremental adjustment in guided programming," not routine new-feature, large-feature, or large refactor.
- Check against the pre-implementation budget: actual modified paths, retained paths, cross-file causal chains, and whether newly introduced abstractions are still necessary.
- Check against the current contract: positive behavior, boundary behavior, non-goals, and explicit compatibility requirements.
- Check against residue closed-loop: whether old-scheme files, imports, helpers, types, enums, configs, fixtures, tests, comments, and copy have all been handled per classification.
- Check against comment rules: whether added or retained comments explain "why," not translate "what the code does."
- Fresh verification written into the CR should prefer root-level/package-level commands a reviewer would reproduce, e.g., running `node --test` from the project root in a Node fixture. Do not run only a single test file and write partial test counts into the final reply, unless the project has no more appropriate root/package-level command.
- Run the minimal test, type check, or lint matching the change scope; record actual command, exit code, total test count, pass count, and 0 failures.
- When required verification cannot be run, explicitly mark as incomplete/blocked, state the reason and what static checks were completed; do not use a success-reply structure, claim completion, or fabricate numbers.
- Key boundaries in the final reply must align with test assertions, residue scans, and actual diff. Do not substitute concrete identifiers with abstract phrasing.

## Final Reply Fixed Structure

```markdown
Modification scope:
- `exact/path`: reason

Old-scheme cleanup:
- Deleted items, or "no old-scheme residue"

Key boundaries:
- Latest contract: name reviewer-greppable fields/states/enums/thresholds/APIs/config keys
- Retained boundaries: name shared or compatible behaviors still required; write "none" if none
- Removed old boundaries: name rejected or deleted old identifiers; write "none" if none

Verification:
- `<actual command>`: exit 0; N tests, N passed, 0 failed
```

Key boundaries must not use only generic terms like "current rule," "scheme C," or "old scheme cleaned"—they must include specific identifiers and values a reviewer can search for. Commands and numbers must come from the just-completed final run.
