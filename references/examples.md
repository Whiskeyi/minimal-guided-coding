# Examples and Anti-Patterns

## Example

User's latest requirement: "The previous scheme A was wrong—change it to check only `status === 'active'`, don't over-modify."

Non-compliant:

```ts
const enableNewStatusFlow = true;

// Compatible with old scheme A and new scheme B
export function isEnabled(item: Item) {
  if (enableNewStatusFlow) {
    return item.status === 'active';
  }
  return item.aPlanFlag && item.status !== 'disabled';
}
```

Compliant:

```ts
export function isEnabled(item: Item) {
  return item.status === 'active';
}
```

If scheme A left behind an `aPlanFlag` type, test, or comment that current logic no longer uses, delete it in sync. Do not retain a dual path to "be compatible with" a scheme the user has already rejected.

## Common Mistakes

| Mistake | Correction |
| --- | --- |
| "Let me extract a shared method—it's more elegant" | Complete the minimal local change first; extract only when real reuse exists |
| "Leaving the old scheme doesn't hurt" | Trace the reference closure and classify; delete residue not needed by the current contract |
| "Keep the old value in tests to prove it's rejected" | Check the current contract first; if it does not explicitly require that exact old input, test the current positive and boundary behavior instead |
| "Fewer files = more minimal" | Minimal means complete contract closure; necessary cross-file boundaries must synchronize |
| "More comments aid understanding" | Only explain non-obvious business reasons—don't translate code |
| "Unify style across a few more files" | Style unification is not the current requirement—raise it separately |
| "Keep both A/B paths just to be safe" | When the latest requirement has rejected A, compatibility is requirement deviation |
| "CR just says 'current rule' / 'scheme C'" | Name specific fields, states, enums, config keys, and old boundaries so a reviewer can search and verify |

## Risk Signals

When these signals appear, pause and shrink the change:

- A small check triggers a new file, cross-directory abstraction, or a large type system.
- Both old and new schemes are supported without user request.
- Only the old production module is deleted, but its identifiers persist in tests, fixtures, comments, or copy.
- For fewer files, necessary callers, state boundaries, or tests of the current contract are omitted.
- The diff contains formatting, sorting, renaming, or file moves unrelated to the current goal.
- Comments explain "what the code does" rather than "why it must be done this way."
- A file's presence in the CR cannot be justified in one sentence.
