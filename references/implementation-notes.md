# Running `implementation-notes.md`

A cross-cutting practice that applies at **every** phase of the loop, from the
moment you start implementing through to ship. Maintain a single
`implementation-notes.md` file alongside the work and update it *as you go* - it
is a running log, not an end-of-task write-up. Writing it at the end defeats the
purpose: the value is capturing each decision while the reasoning is fresh, so
the person reviewing (and future-you) can see *why* the build looks the way it
does, not just what it does.

## Why it matters

When an agent implements a spec, the interesting information is everywhere the
build *didn't* mechanically follow the spec: the ambiguities it resolved, the
places it knowingly departed, the alternatives it weighed, and the things it's
unsure about. That information normally evaporates. This file captures it so the
human can confirm or correct course quickly, and so the review phase has the
context it needs.

## What to capture

Maintain four sections and append to them throughout:

- **Design decisions** - choices made where the spec was ambiguous or silent.
 State what you chose and the reasoning.
- **Deviations** - places where you intentionally departed from the spec, and
 why. (A deviation you can't justify in writing is a bug, not a decision.)
- **Tradeoffs** - alternatives you considered and why you picked what you did.
- **Open questions** - anything you'd want the spec owner to confirm or revise.
 Surface these early, don't sit on them.

## Template

```markdown
# Implementation notes - <feature / spec name>

_Running log. Updated as the build progresses, not written at the end._

## Design decisions
- <ambiguity in spec> → chose <X> because <reasoning>.

## Deviations
- Spec said <Y>; implemented <Z> instead because <reasoning>.

## Tradeoffs
- Considered <A> vs <B>; picked <A> because <reasoning>; <B>'s cost was <…>.

## Open questions
- [ ] <question for the spec owner to confirm or revise>
```

## How it connects to the rest of the loop

- **Phase 2 (implementation):** the notes file is born here and grows with every
 non-obvious choice.
- **Phase 3 (review):** hand the notes to the reviewer alongside the diff - the
 "Open questions" and "Deviations" sections are where review effort pays off
 most.
- **Phase 4 (ship):** unresolved open questions either get answered before
 shipping or are recorded in the commit body as accepted.
