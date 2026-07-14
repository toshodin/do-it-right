# do-it-right - the 13-step build loop

A [Claude Code](https://docs.anthropic.com/en/docs/claude-code) skill that turns
"build this feature" into a disciplined loop: **plan → implement → review → test →
ship**, with a hard gate that nothing ships until an *independent* review comes
back clean, a saved end-to-end test is in the suite and green, and - for anything a
user can see - the real flow has been driven in a browser against the deployed app.

## Why it exists

Passing type-checks, the build, and a wall of unit tests is **not the same** as
being correct. This loop was created after a single independent review pass caught
seven real bugs - including a critical privacy leak - in code that had already gone
green on all of the above. Review and end-to-end testing are treated as part of
*building*, not an optional afterthought.

## The one rule

A unit of work is **not done** until all three are true:

1. An **independent review** (a subagent in a fresh context, or a *different* model
 via an external API) returns no medium-or-higher findings - or each remaining
 finding is explicitly accepted with a recorded reason in the commit body. The
 agent never marks its own homework.
2. The feature's **saved end-to-end test is in the suite and green.**
3. For user-facing behaviour: you have **driven the real flow in a browser against
 the deployed app and watched it work.** Deployed ≠ verified.

## The phases

| Phase | Steps | File |
|---|---|---|
| Plan vetting | 1-3 | [`references/phase-1-plan-vetting.md`](references/phase-1-plan-vetting.md) |
| Implementation | 4 | [`references/phase-2-implementation.md`](references/phase-2-implementation.md) |
| Review | 5-10 | [`references/phase-3-review.md`](references/phase-3-review.md) |
| Ship | 11-13 | [`references/phase-4-ship.md`](references/phase-4-ship.md) |
| Bug path (replaces 1-2 for bugs) | - | [`references/diagnosing-bugs.md`](references/diagnosing-bugs.md) |
| Running notes (cross-cutting) | - | [`references/implementation-notes.md`](references/implementation-notes.md) |

The skill loads only the phase you're working on, not the whole loop at once, and
keeps a running `implementation-notes.md` alongside the work so no review or test
phase gets skipped and every decision is recorded.

## Install

Copy the folder into your Claude Code skills directory:

```bash
git clone https://github.com/<your-username>/do-it-right.git ~/.claude/skills/do-it-right
```

It triggers automatically on build / implement / fix / "make it faster" work, or
call it explicitly with `/do-it-right`.

## Adapting it to your stack

The loop is deliberately tool-agnostic. Map these seams to your own setup:

- **Independent reviewer** - a `code-reviewer` subagent, or any second-opinion
 model reached over an API. Independence from the author's context is the point.
- **Simplify pass** - Anthropic's `code-simplifier` agent, or the checklist in
 Phase 3 applied by hand.
- **Deploy & verification path** - recorded per change in the spec, because it's
 project-specific (preview→promote, CDK deploy, auto-build on merge, etc.).
- **Team knowledge store** - the "tribal knowledge" review lens grounds in a shared
 wiki / ADR log / decision store if you keep one. Optional.
- **Review rule set** - the review gate scores against a written set of rules. The
 skill ships a generic starter set on purpose. Grow your own (see below).
- **Real-browser verification (gate 3)** - a real deployed, authenticated session you watch end to end (e.g. the Claude in Chrome extension), not a headless run or one specific tool. The point is a human-observed live flow.

## Build your own review rule set

The review gate (Phase 3, Step 9) is only as good as the rules it scores against,
and the most valuable rules are not generic best-practice lists. They are the
specific corrections *your* reviews keep making. So this skill ships a deliberately
generic starter set for you to replace, not adopt.

The method is to harvest your own agent (and human) review sessions:

1. **Capture.** After each review, keep the findings your reviewer raised, whether
 that reviewer is a person or a second-opinion model. The signal you want is
 repetition: the same correction showing up across different changes.
2. **Cluster.** Group recurring findings into themes (simplicity, single source of
 truth, verification, naming, and so on).
3. **Distil.** Write each theme as a short rule with a stable ID and a one-line
 example of the smell and the fix.
4. **Feed back.** Hand that rule set to the Step 9 reviewer so future reviews cite
 your rules by ID. Keep it in version control next to the code.
5. **Compound.** Every review sharpens the set. Rules that never fire get dropped,
 new patterns get added.

**Harvesting a specific expert.** If one reviewer on your team is consistently
sharp, you can accelerate this by collecting their past review comments and
distilling them into rules. Do it with their blessing, and keep the attribution:
their rules are their contribution, not yours. Distil the practice, credit the
person.

This is the same loop the skill applies to code, turned on the review itself:
capture what a fresh pair of eyes catches, make it durable, and let it compound.

## Credits

The implementation and bug-path phases adapt structure and discipline from
[Matt Pocock's skills](https://github.com/mattpocock/skills) (`to-prd`, `tdd`,
`diagnosing-bugs`, `to-issues`). Security review is anchored on the
[OWASP](https://owasp.org/) ASVS and Top 10.

## Kudos

Thanks to [Andrius Kr](https://github.com/andrius-kr) for helping review and refine
the concept beyond its initial version.

## Licence

MIT - see [LICENSE](LICENSE).
