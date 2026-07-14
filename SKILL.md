---
name: do-it-right
description: >-
 The 13-step build loop (a.k.a. "the 13-step loop" or "do it right"): plan →
 implement → review → test → ship, with a hard gate that nothing ships until a
 subagent or external-API review comes back clean and the feature has a saved
 end-to-end test in the suite. Use this whenever you are about to build,
 implement, or change any non-trivial feature, fix a bug, or modify behaviour -
 even if the user doesn't name the loop. Trigger on "build a feature",
 "implement X", "add X", "fix this bug", "make it faster", "do it right", "the
 loop", "13 step loop", "follow the loop", or before committing/shipping a
 user-facing change, a new endpoint, or an access/permission change. Pulls each
 phase into a tracked checklist, loads only the phase you're working on, and
 keeps a running implementation-notes.md so no review or test phase gets skipped
 and every decision is recorded.
---

# The 13-Step Loop ("Do It Right")

A build discipline for shipping features and fixes that hold up. It exists
because passing type-checks, the build, and a wall of unit tests is **not the
same** as being correct: in the incident that created this loop, a single
independent review pass caught seven real bugs - including a critical privacy
leak - in code that had already gone green on all of the above. Review and
end-to-end testing are part of *building*, not an optional pass afterwards.

## The one rule that matters

A unit of work is **not done** until ALL THREE of these are true:

1. A **subagent or external-API review** (a genuinely different context - see
 below) returns **no medium-or-higher findings** - or each remaining finding is
 explicitly accepted with a recorded reason in the commit body.
2. The feature's **saved end-to-end test is in the suite and green.**
3. For any user-facing behaviour: you have **driven the real user flow in a
 browser, against the deployed app, and watched it work** - not inferred it from
 unit tests, an automated spec, or a green deploy. Deployed ≠ verified;
 unit-tested ≠ verified. You confirm with your own eyes that the actual flow
 (navigate → act → observe the outcome the user would see) behaves as expected,
 AFTER it's deployed to the target environment. **Always use your project's real-browser verification tool** (e.g. the Claude in Chrome extension, `mcp__Claude_in_Chrome__*`) for this - not Playwright runs,
 the preview server, or a unit/integration stand-in. If you cannot run the
 browser e2e, the work is **not complete** - say so explicitly and do not claim
 it is. Never substitute "tests pass + it's deployed" for having watched it.

Everything in the phases serves those three gates. The classic failure this guards
against: building, unit-testing, deploying, and declaring "complete" without ever
opening the app - passing tests and a clean deploy are necessary, not sufficient.

## Before you type the word "done" (the pre-flight)

The three gates fail most often not because the words are missing but because the
actions get skipped under time pressure. So make them literal, every single time:

1. **Run the saved suite NOW. Do not trust that it "would pass".** In a real
 incident on this stack, a route-smoke e2e *already covered* the screen that
 crashed. It was never run before "done" was declared, so the crash shipped to the
 user and they found it, not us. Green-in-principle is not green. Run it.
2. **Open the deployed app NOW and walk the exact flow the user will.** Not a
 Playwright run, not the preview, not "the deploy succeeded". Your own eyes on the
 real thing, after deploy, via your real-browser verification tool (e.g. the Claude in Chrome extension). If you genuinely cannot (an
 upload is blocked, no access), say so plainly and mark the work NOT verified.
 Never round "I couldn't watch it" up to "done". A deterministic proof (e.g. an
 RLS simulation for a policy change) can stand in ONLY where a browser cannot
 exercise the behaviour, and only when it exercises the real object, not a mock.
3. **e2e is part of implementation, not a follow-up.** A feature is not "built, and
 next I'll test it". It is not built until it is verified this way. Do not report a
 user-facing change as done, complete, or ready to try until gates 2 and 3 have
 actually been executed and their result stated.

## What "subagent or external-API review" means

The reviewer must **not be the context that wrote the work** - self-review by the
same agent reliably misses its own blind spots. A quick fresh-eyes self pass is
worth doing first because it's cheap and catches the obvious, but it is *not* the
gate. The gate is review by a different context:

- a **subagent** (e.g. a `code-reviewer` sub-agent in a clean context), or
- an **external API** - a *different model* than the one that wrote the code
 (e.g. a review hook calling Gemini, or any second-opinion model).

Independence is the point, not the specific tool. Map these to your stack.

## Cross-cutting: keep a running `implementation-notes.md`

From the moment you start implementing until you ship, maintain an
`implementation-notes.md` alongside the work. It is the audit trail of how the
build diverged from or interpreted the spec - **update it as you go, at every
phase, not as a write-up at the end.** It captures design decisions, deviations,
tradeoffs, and open questions. Full template and rules:
**[references/implementation-notes.md](references/implementation-notes.md)**.

## How to use this

Create one tracked todo per phase below. When you reach a phase, **read its
reference file** - don't try to hold the whole loop in context at once. Each file
is self-contained for that phase. Walk the phases in order. The boundaries are
real (don't implement before the plan is vetted, don't ship before review is
clean).

**Routing first:**
- **A bug** (something broken / throwing / failing / slow)? Start with the
 **[bug path](references/diagnosing-bugs.md)** instead of Phases 1-2 - it produces
 a tight repro, the fix, and a regression test, then rejoins at Phase 3.
- **A large piece of work?** Atomise it into bite-size vertical slices first (the
 [`to-issues`](https://github.com/mattpocock/skills) pattern), then run *each slice* through the loop.

## The phases

1. **Plan vetting** - spec it (PRD template + test seams + deploy path), then get
 the *plan* reviewed through the persona lenses before any code.
 → **[references/phase-1-plan-vetting.md](references/phase-1-plan-vetting.md)** (steps 1-3)
2. **Implementation** - test-first, vertical red-green-refactor at the behaviour
 layer (Pocock `tdd` discipline). Flow-level e2e deliberately deferred.
 → **[references/phase-2-implementation.md](references/phase-2-implementation.md)** (step 4)
3. **Review** - the part people skip. An OWASP-anchored persona panel (security /
 privacy / correctness / domain lenses), looping until the reviewer is clean.
 → **[references/phase-3-review.md](references/phase-3-review.md)** (steps 5-10)
4. **Ship** - crystallise the e2e, commit (Conventional Commits), verify deploy.
 → **[references/phase-4-ship.md](references/phase-4-ship.md)** (steps 11-13)

**Bug path** (replaces Phases 1-2 for bugs) - feedback loop → reproduce/minimise →
3-5 ranked hypotheses → instrument → fix + regression test, then rejoin at Phase 3.
→ **[references/diagnosing-bugs.md](references/diagnosing-bugs.md)**

## When you can skip the review and simplify steps

Steps 8-10 are skippable **only** for changes that can't carry the risks they
guard against: doc-only commits, reverts, pure dependency bumps, and production
hot-fixes (which must be reviewed *after the fact*, reason in the commit body). If
you're reaching for this list to avoid reviewing actual feature code, that's the
signal you most need the review.
