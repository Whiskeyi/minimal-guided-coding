# Scheme-Switch Cleanup, Implementation-Write Rules, and Comment Rules

## Scheme-Switch Cleanup

Treat a rejected scheme as a reference network that must be closed out, not just a few production files:

- Starting from known files, fields, and values, search for exact identifiers across imports, helpers, types, enum values, config keys, fixture fields, tests, comments, and user-facing copy.
- Classify each hit as `delete`, `retain`, or `restore`, and record the reason.
  - `Retain`: Shared code still depended on by the latest behavior or an explicit compatibility requirement.
  - `Restore`: Code broken by the old scheme that the current contract requires to revert to its original behavior.
- "replace/remove/rename old X to new Y" describes only the substitution relationship—it does not by itself require permanently retaining test cases containing X in the final tests.
- Only retain an exact old input in final tests when the latest contract explicitly specifies expected behavior for that precise old input; otherwise test the new value and a generic invalid or boundary value.
- If proof that the old value is rejected is needed, use a temporary test or temporary probe to confirm; do not retain the superseded old value in final project tests, fixtures, comments, or copy—avoiding re-solidifying the old scheme into the CR.
- During implementation, temporary cleanup tests may be used to prove a file, field, or symbol has been deleted; after residue scanning is complete, archival-nature tests must be removed from the final project.
- When no test was specified and similar code in the repo also has no tests, temporary validation tests/probes must be deleted before final delivery.

A complete closed-loop search discovers hidden residue scattered in fixtures, copy, and tests; per-item classification prevents accidental deletion of shared implementations still used by current behavior.

## Implementation-Write Rules

The agent is responsible for completing the change. Project files by default carry only business implementation, necessary tests, and real copy—avoid writing user raw instructions, task context, or "per user request" meta-descriptions into code, comments, fixtures, test names, or user-facing copy. When a special encoding genuinely requires explanation, a single business-phrased modification-motivation sentence may describe the reason without echoing conversation instructions.

## Comment Rules

Comments are budget items, not courtesy items.

### Allowed to Add or Retain

- A business rule, compatibility constraint, or boundary condition that is not obvious from code structure.
- Logic complex enough that removing the comment would prevent a reviewer from judging why it was written this way.
- A type, enum, or public constant carrying business semantics that requires explanation of value meaning or external contract.
- A limitation that cannot be eliminated for now and needs to note the reason and future handling condition.

### Must Delete

Comments where the code logic is clear, naming already conveys intent, and the comment merely translates what the code does. Do not add comments to obvious branches, simple assignments, straightforward functions, or local variables just to make the CR look "more solid."
