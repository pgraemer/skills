---
description: How I want code written, tested and verified
---

# Coding

## Ownership

You own the code you write, the way a maintainer does: you are the one who
has to read, debug and change it later. "It works" is not done.

* **Reuse before you write.** Search for what already exists and use or
  extend it. Don't add a second helper, type or component that does what an
  existing one does.
* **Leave nothing you wouldn't want to inherit.** No unused code, leftover
  debug output, fallbacks for cases that can't happen, or abstractions with
  a single use.
* **Finish the change.** When your change makes other code redundant or
  wrong (callers, tests, docs), fix that in the same change.
* **Read your whole diff before you finish.** Remove everything the task
  doesn't need, including CSS declarations that don't change the result.

## Code

* **Comments:** Comment only what the code can't say itself (non-obvious
  rationale, business logic), in two sentences or fewer.
* **TypeScript:** Avoid `as` assertions. Use type guards, narrowing,
  inference or generics.

## Testing and verification

* **Verify once, at the end.** Don't run type checks, linters or tests after
  each edit. Finish the implementation, update the tests, then run the full
  verification.
* **Test data:** In backend code, build fixtures and mocks with factories
  that have sensible defaults. Don't hardcode mock objects in individual
  tests.
* **Review skill:** If the repository has a review skill, run it as the
  last step.

## Shell

* Don't prefix commands with `cd`. Use absolute paths or the tool's
  working-directory option.
