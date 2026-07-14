# Phase 2 - Implementation

## Step 4 - Code + unit tests, test-first, in vertical slices

Build the feature **one behaviour at a time, test-first**. This step adopts the
discipline of Matt Pocock's [`tdd` skill](https://github.com/mattpocock/skills)
(`mattpocock/skills`, model-invoked) for the unit layer - if that skill is
installed, defer to it. The rules below are the distilled version so this loop is
self-contained without it.

### When: vertical red-green-refactor (never horizontal)

Work in **vertical slices** - one test, then the code to pass it, then the next -
not all tests first then all code:

```
RIGHT (vertical):           WRONG (horizontal):
  RED→GREEN: test1→impl1       RED:   test1, test2, test3, ...
  RED→GREEN: test2→impl2       GREEN: impl1, impl2, impl3, ...
  RED→GREEN: test3→impl3
```

Horizontal slicing produces **crap tests** - written in bulk against *imagined*
behaviour, they verify the *shape* of data structures rather than user-facing
capability, and they pass when behaviour breaks. Avoid it.

1. **Planning.** Confirm the public interface you're building and **prioritise
 which behaviours matter** - you can't test everything, so name the critical
 paths and complex logic, not every edge case. List behaviours to test, not
 implementation steps.
2. **Tracer bullet.** Write ONE test for ONE behaviour → watch it fail → write the
 minimal code to pass. This proves the path end-to-end.
3. **Incremental loop.** For each remaining behaviour: RED (next test fails) →
 GREEN (minimal code passes). One test at a time. Only enough code to pass the
 current test. Don't anticipate future tests.
4. **Refactor - only once GREEN.** Extract duplication, deepen modules (complexity
 behind a small interface), run tests after each step. **Never refactor while
 RED.**

### What: behaviour through the public interface

A good test **reads like a specification** - `user can checkout with valid cart`
tells you the capability exists. It exercises real code paths through the public
API and **survives refactors** because it doesn't care about internal structure.

Per-cycle checklist:

- [ ] Test describes **behaviour**, not implementation
- [ ] Test uses the **public interface** only
- [ ] Test would **survive an internal refactor**
- [ ] Code is **minimal** for this test
- [ ] **No speculative** features added

The warning sign of a bad test: it breaks when you rename an internal function or
refactor, even though behaviour hasn't changed. Don't mock internal collaborators,
test private methods, or verify through the back door (e.g. querying the DB
directly instead of calling `getUser`).

### The flow-level e2e stays deferred - and that's consistent

Test-first applies to the **behavioural/contract** layer above: those tests are
decoupled from implementation, so they *don't churn* when review later changes the
flow. The **flow-level e2e** (e.g. Playwright driving the rendered UI) is a
*different* artefact - it couples to the final flow and so is deliberately deferred
to **step 11**, after review has settled the behaviour. Writing it now risks churn;
that's the only thing you defer, not the behavioural tests.

> Exercise the real user-facing flow as you build (browser, CLI, wherever it
> lives) as **throwaway probing** - it informs the step-11 e2e but isn't saved yet.

## Maintain `implementation-notes.md` from here on

This is where the running notes file is born and earns its keep. Every non-obvious
choice - a resolved spec ambiguity, a deliberate deviation, a tradeoff, a question
for the spec owner - gets logged immediately under the right heading. Don't batch
it for later. The reasoning is freshest now. Full template:
[implementation-notes.md](implementation-notes.md).

→ Next: [phase-3-review.md](phase-3-review.md)
