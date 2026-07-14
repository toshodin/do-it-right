# Phase 4 - Ship

Only enter this phase once Phase 3's gate is clean (review returns no medium+
findings, or each is explicitly accepted).

## Step 11 - Now write the saved end-to-end test

Behaviour is final, so lock it in. Write a happy-path render plus the key
interaction, and - for anything gated - the negative/denied case too. Keep these
tests durable:

- reuse a stored session instead of logging in per test;
- assert on seeded data or specific strings, never on global counts that other
 tests can shift;
- give create/mutate tests unique names and have them clean up after themselves;
- group steps that depend on each other serially. Keep independent checks
 parallel.

Run the suite green. **Don't ship a user-facing flow, a new endpoint, or an
access/permission change without its saved end-to-end test plus unit coverage** -
the suite grows with the product, so the next person's change can't silently
break yours.

## Step 12 - Commit and push

One coherent phase per commit. Use **Conventional Commits** - `type(scope):
summary`, where `type` is `feat`, `fix`, `chore`, `refactor`, `docs`, `test`,
`perf`, etc. - with a **body that describes the diff**: what changed and why. If
you accepted any review finding rather than fixing it (step 10), record the reason
in the body. If this was a bugfix, state the correct hypothesis there too. Resolve
or explicitly accept any remaining `implementation-notes.md` open questions before
committing - don't ship unanswered questions silently.

## Step 13 - Verify the deployment, AND drive the real flow in a browser

Execute the **deployment & verification path** you captured in the spec (Phase 1,
step 1). It's project-specific by design - Vercel preview→promote, AWS CDK deploy,
auto-build on merge to main, etc. - which is why it's recorded per change rather
than hard-coded here. Whatever the mechanism:

- **Watch error rates / logs** for a beat after it lands.
- **Know the rollback** - how to revert if the smoke-check fails.

Then the non-negotiable part for any user-facing behaviour (gate 3):

- **Open the deployed app in a browser and drive the actual user flow yourself** -
 navigate to the feature, perform the steps a user would (pick, fill, submit,
 download, etc.), and **observe the real outcome on screen**. A green build, a
 passing unit/e2e suite, and a successful deploy are necessary but **not**
 verification - you have not verified a feature until you've watched it work.
- **Use your project's real-browser verification tool** (e.g. the Claude in Chrome extension, `mcp__Claude_in_Chrome__*`) for this - driving
 the live app with a real authenticated session. Not Playwright, not the preview
 server, not a DB-level stand-in. Reuse a logged-in session. Don't enter
 credentials yourself.
- **If you genuinely cannot run the browser e2e** (env down, auth unavailable, the
 flow needs data you can't seed): the work is **NOT complete**. Say so plainly,
 list exactly what's unverified and why, and treat the browser e2e as owed before
 the feature can be called done. Do **not** report it as complete.

If you're batching rather than pushing live now, the browser e2e is owed at the
next deploy - and "complete" is not claimed until it's done.

→ Loop complete only when ALL THREE gates are met: review clean, saved end-to-end
test green, **and the real user flow watched working in a browser on the deployed app.**
