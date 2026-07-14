# The bug path - diagnosing-bugs

For tasks that are **broken / throwing / failing / slow**, not features. Run this
*instead of* Phases 1-2. It produces the fix plus a regression test, then rejoins
the loop at **Phase 3 (review)**. Distilled from Matt Pocock's
[`diagnosing-bugs`](https://github.com/mattpocock/skills) skill - skip a phase only
with explicit justification.

## 1. Build a feedback loop - *this is the skill*

Everything else is mechanical. With a **tight pass/fail signal that goes red on
this specific bug**, bisection and hypothesis-testing just consume it. Without one,
no amount of staring at code will save you. Spend disproportionate effort here. Be
aggressive, be creative, refuse to give up.

Ways to construct one, roughly in order: a **failing test** at the seam that
reaches the bug → a **curl/HTTP script** against a dev server → a **CLI invocation**
diffing output → a **headless browser script** → **replay a captured trace** → a
**throwaway harness** → a **property/fuzz loop** → a **bisection harness** (`git
bisect run`) → a **differential loop** (old vs new). Human-in-the-loop clicking is
the last resort.

**Then tighten it** - treat the loop as a product: faster (narrow scope, skip
unrelated init), sharper (assert the *exact symptom*, not "didn't crash"), more
deterministic (pin time, seed RNG, isolate FS/network). For non-deterministic bugs
the goal isn't a clean repro but a **high enough reproduction rate** to debug
against (loop 100×, parallelise, add stress).

**Completion criterion:** one command you have **already run** (paste the
invocation + output) that is red-capable (catches *this* bug, asserts the user's
exact symptom), deterministic, fast, and agent-runnable. **No red-capable command,
no hypothesising** - jumping to a theory before the loop exists is the exact failure
this prevents.

## 2. Reproduce + minimise

Run the loop, watch it go red. Confirm it's the failure the **user** described (not
a nearby one - wrong bug = wrong fix). Then shrink to the **smallest scenario that
still goes red**: cut inputs, callers, config, data, steps **one at a time**,
re-running after each cut. Done when every remaining element is load-bearing. A
minimal repro shrinks the hypothesis space and becomes the clean regression test.

## 3. Hypothesise

Generate **3-5 ranked, falsifiable hypotheses** before testing any - single-
hypothesis thinking anchors on the first plausible idea. Each must state a
prediction: "If X is the cause, then changing Y makes the bug vanish." No
prediction = a vibe. Discard or sharpen it. Show the ranked list to the user before
testing - they often re-rank instantly ("we just deployed #3"). Don't block if
they're away.

## 4. Instrument

Each probe maps to a specific prediction. **Change one variable at a time.** Prefer
a debugger/REPL (one breakpoint beats ten logs). Else targeted logs at the
distinguishing boundaries - never "log everything and grep". **Tag every debug log**
with a unique prefix (`[DEBUG-a4f2]`) so cleanup is one grep. For performance
regressions, logs are usually wrong - measure a baseline (profiler, query plan,
`performance.now()`) first, then bisect.

## 5. Fix + regression test

Write the regression test **before the fix** - but only if a **correct seam**
exists (one that exercises the real bug pattern at the call site). A too-shallow
seam gives false confidence; **if no correct seam exists, that itself is the
finding** - note it, the architecture is preventing lockdown. With a correct seam:
turn the minimised repro into a failing test → watch it fail → apply the fix →
watch it pass → re-run the original (un-minimised) loop.

## 6. Cleanup + post-mortem

Before declaring done: original repro no longer reproduces. Regression test passes
(or absence of seam is documented). All `[DEBUG-…]` instrumentation removed
(grep the prefix). Throwaway harnesses deleted. **State the correct hypothesis in
the commit/PR** so the next debugger learns. Then ask: *what would have prevented
this?* - if the answer is architectural (no seam, tangled callers, hidden
coupling), flag it.

→ Rejoin the loop at **[phase-3-review.md](phase-3-review.md)** (the fix gets the
same persona review as any change), then **[phase-4-ship.md](phase-4-ship.md)**.
