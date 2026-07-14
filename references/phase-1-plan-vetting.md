# Phase 1 - Plan vetting (before any code)

Vet the design on paper. Fixing a flaw here is an order of magnitude cheaper than
fixing it in shipped code.

> **Is this a bug, not a feature?** If the task is something broken / throwing /
> failing / slow, don't plan a feature - run the **[bug path](diagnosing-bugs.md)**
> instead. It produces a tight repro, the fix, and a regression test, then rejoins
> this loop at Phase 3 (review).

> **Is this a large piece of work?** Atomise it *before* the loop. Break the
> product / epic into bite-size, independently-shippable **vertical slices** (the
> [`to-issues`](https://github.com/mattpocock/skills) pattern), each with its own acceptance criteria, then run *each
> slice* through this loop. Smaller slices = trackable progress and reviews that
> stay in one context.

## Step 1 - Write the spec

Use a consistent template so specs are reviewable and comparable. This adapts Matt
Pocock's [`to-prd`](https://github.com/mattpocock/skills) structure:

<spec-template>
## Problem
The problem the user faces, from the user's perspective.

## Solution
The solution, from the user's perspective.

## User stories
A numbered, extensive list - "As a `<actor>`, I want `<feature>`, so that
`<benefit>`" - covering all aspects of the feature.

## Implementation decisions
Modules touched/added, the interfaces that change, schema changes, API contracts,
architectural decisions. **Describe modules and interfaces, not file paths or
code** - paths go stale the moment you refactor.

## Testing decisions - the seams
Name the **seams** you'll test through (the boundaries/interfaces a test can
drive), not a file list. Prefer existing seams. Use the **highest** seam (most
user-facing) you can. The **fewer seams the better - ideally one**. A good test
here exercises behaviour through the public interface, not implementation. This
doubles as the Phase 2 TDD plan.

## Deployment & verification path
How this ships and how you'll confirm it took - discovered as part of grounding,
because it's project-specific (Vercel preview→promote, AWS CDK deploy, auto-build
on merge to main, etc.). Captured here so **step 13** has a concrete thing to run.

## Success criteria
Verifiable, not vague: "returns 403 for non-owners", not "handles permissions".

## Security & privacy
What data is touched, what could leak, which access rules apply. Ground this in any
static-analysis or attack-surface inventory you have for the repo (SAST output, an
endpoint/route map, prior security findings) so the threat model starts from the
real surface, not guesswork.

## Out of scope
What you are deliberately **not** doing. The line that stops scope creep.
</spec-template>

## Step 2 - Quick self pass (cheap, not the gate)

Read the plan adversarially, as if trying to break it: What threat model does it
introduce? What did you assume without checking? What's missing? Where could it
collide with existing schema, access rules, or consent flows? Fix the obvious
holes now so the real review isn't spent on them. This pass is cheap and worth
doing - but it is **not** the gate.

## Step 3 - Subagent or external-API review of the plan (the gate)

Hand the spec to a **different context** and review it through the *same persona
lenses* you'll use on the code (Phase 3), but at plan altitude - catching a design
flaw here is the cheapest fix there is:

- **🔒 Security-by-design / threat model** - what's the attack surface? Where could
 broken access control, data exposure, or injection enter *by design*? (OWASP A04
 Insecure Design - shift left.)
- **🕵️ Privacy & consent** - does the data model collect or expose more than it
 needs? Consent and entitlement gaps.
- **🐛 Correctness & coverage** - missed edge cases, gaps in the user stories,
 success criteria that aren't actually verifiable, missing test seams.
- **Scope** - scope-creep risk. Is the out-of-scope list honest?

Resolve every medium+ finding before writing code.

## Notes

Any spec ambiguity you spot here is the first "open question" to log once you begin
Phase 2. See [implementation-notes.md](implementation-notes.md).

→ Next: [phase-2-implementation.md](phase-2-implementation.md)
