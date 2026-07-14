# Phase 3 - Review (the part people skip - don't)

This phase contains the gate. A quick self pass comes first because it's cheap,
but the **subagent or external-API review** (step 9) is what actually clears the
work.

## Step 5 - Quick self pass, fresh eyes (cheap, not the gate)

Read the whole diff as if someone else wrote it.

## Step 6 - Quick self pass against the spec (cheap, not the gate)

Read the diff *with the Phase 1 spec open beside it* and tick off every success
criterion. Anything unticked is unfinished. Steps 5-6 are cheap and catch the
obvious, so the real review isn't wasted on them - but they are **not** the gate.

## Step 7 - Local checks

Type-check + build + the targeted unit tests on the files you touched. If the
change is user-facing or touches access/permission rules, also run the *existing*
end-to-end suite against the local stack as a regression check.

## Step 8 - Simplify pass

Run a dedicated simplification pass over the new/changed code (e.g. Anthropic's
**`code-simplifier`** agent if you have it), then **re-run step 7**. Order matters:
simplify *after* tests are green (they're the safety net proving behaviour is
preserved) and *before* the review (step 9) so the reviewer sees the final shipped
form, not a draft. Reject any simplification that turns the checks red.

It targets: dead code, duplication, over-abstraction, **shallow modules** (push
complexity behind a small interface - favour deep modules), inconsistent naming,
and comments that merely narrate what the code plainly says (keep only the *why*,
2 lines max). Without such an agent, this list *is* the discipline - apply it by
hand.

## Step 9 - Subagent or external-API review (the gate)

Review by a **different context** - a `code-reviewer` sub-agent in a clean
context, or an external-API review by a *different model* than the one that wrote
the code. The author's own context can't reliably see its own blind spots, which
is the whole reason this step exists.

Always hand over the diff **and the `implementation-notes.md`** - the "Deviations"
and "Open questions" sections are where review effort pays off most.

### The rubric: cite a rule ID, use a severity model

A review is sharper when it scores against a **written rule set** than when it
rides on a reviewer's mood. Give every rule a **stable ID** and cite it in each
finding, so findings are comparable across reviews and you can see which rules fire
most often.

Don't inherit someone else's list. **Grow your own** from the corrections your
reviews actually surface (the README shows the [method](../README.md#build-your-own-review-rule-set)). The set below is a
deliberately generic starter, shown only for the *shape* of it. Replace it with
yours:

- `SIMPLE-n`: the simplest thing that works, no abstraction a caller can't justify.
- `DRY-n`: one source of truth, fix the cause not each downstream symptom.
- `CORRECT-n`: edge cases, nulls, races, adversarial inputs.
- `SCOPE-n`: change only what the task needs, watch the blast radius.
- `TEST-n`: tests exercise the real path and assert real values, not shape.
- `VERIFY-n`: claims backed by evidence, failures loud rather than swallowed.
- `ARCH-n`, `NAME-n`, `UX-n`: architecture, naming, user-facing safety.

Keep the set in version control so every reviewer, human or agent, scores against
the same thing.

**Run a fast first pass** on the highest-leverage checks before the detail: is the
fix at the source rather than patched downstream? Is it the smallest change that
resolves the root cause? One code path, one source of truth? Do the tests exercise
real behaviour and assert real values? Is every claim backed by evidence, and does
it fail loud?

**Severity to gate mapping:** **Blocker** and unjustified **Major** are the gate.
Loop 9 to 10 until they are clear, or each Major carries a written justification in
the commit body. **Minor** is advisory, raise it but don't block on it. In practice
the two themes that catch the most real problems are *simplicity fixed at the
source* and *genuinely verified work*, so weight those highest.

### Review personas - each a distinct lens with a standard

Don't ask one reviewer to "check it's secure." Run the **relevant personas**, each
with its own standard and checklist - different lenses catch what a single generic
pass misses. For a large or sensitive change, run them as **parallel subagents**;
for a small one, a single reviewer steps through the relevant checklists. Every
persona returns confidence-rated, file-and-line-specific findings.

**Always-on** for any change touching auth, data, endpoints, or user-facing behaviour:

- **🔒 Security** - *verify against* **OWASP ASVS 5.0 at Level 2** (scale to L3 for
 payments, health, or other high-sensitivity data. L1 only for low-risk internal
 tools), and *report findings by* **OWASP Top 10:2025 category** (A01 Broken
 Access Control - now includes SSRF - through A10 Mishandling of Exceptional
 Conditions. Pull the authoritative current list from
 <https://owasp.org/Top10/2025/>, don't rely on memory). Conditional add-ons:
 **OWASP API Security Top 10** when there's a new/changed endpoint; **OWASP Top 10
 for LLM Applications** when the change touches AI prompts or model I/O. Back it
 with tools, not just model judgement, where you can - e.g. a `semgrep` scan, a
 database RLS/auth audit, a dedicated security-review pass.
- **🕵️ Privacy & consent** - data minimisation, PII handling and retention, consent
 flows, and **access-control / row-scoping leaks** (the entitlement failures unit
 tests sail past - this is the lens that catches the leak that started this loop).
- **🐛 Correctness / adversarial** - adversarial inputs, race conditions, schema/null
 mismatches, regex backtracking, pattern-matching false positives, and anything
 that only breaks under load or at the boundaries.

**Fire when the change touches their domain:**

- **⚡ Performance & scale** - N+1 queries, unbounded loops/allocations, missing
 pagination, hot-path cost.
- **♿ Accessibility** - semantic markup, keyboard navigation, focus management,
 ARIA, colour contrast (UI changes).
- **🗄️ Data integrity & migrations** - reversibility, backfill safety,
 nullability/constraints, ordering, zero-downtime (schema/migration changes).
- **🧠 Tribal knowledge / team conventions** *(fire when your team keeps a shared
 knowledge store - a wiki, an ADR log, a decision or gotcha record)* - **before**
 reviewing, ground in it: pull the touched area's known gotchas, its invisible
 contracts (undocumented behaviours other components rely on), and any cross-
 component integration edges for a change that spans boundaries. Then review the
 diff *against* that grounding: does it violate a known gotcha, break an invisible
 contract, or contradict a past decision? And **fact-check any automated/AI review
 flag against that source of truth** - this lens replaces hallucinated cross-repo
 review with the team's real, recorded knowledge. Corrections it surfaces should
 feed back into the store so the next review inherits them.

> **Plan-vetting tie-in (Phase 1):** if you keep a team knowledge store, do the same
> pull *before* writing code - surface the touched area's gotchas and contracts so
> the plan doesn't design something that violates a known contract in the first place.

## Step 10 - Fix every medium+ finding, add a regression test for each, loop

Fix each medium-or-higher finding and add a unit test for the catch (a catch
without a regression test will come back). Then **loop back to step 9** and review
again. Keep looping 9↔10 until the review returns clean. **This loop is the gate**
 - don't move on because you're tired of it.

→ Next: [phase-4-ship.md](phase-4-ship.md)
