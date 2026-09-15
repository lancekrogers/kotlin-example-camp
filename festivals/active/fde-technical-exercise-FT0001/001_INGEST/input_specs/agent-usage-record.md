# Agent usage so far — raw material for `AGENT_WORKLOG.md` (brief §5)

> **Why this is an input spec:** brief §5 requires an `AGENT_WORKLOG.md` in the fork covering
> harnesses used, representative instructions, where agents changed the approach, how their work
> was verified, and **something they got wrong**. That material exists only in the session history
> of the security-audit work that produced `bf1435e`. Captured here so the festival can draw on it
> instead of reconstructing it from memory later. Raw notes, not the finished worklog.

## Harness and models

- **Claude Code** (CLI), model Opus 5, 1M context. Single harness, multiple sessions.
- Camp/Festival methodology tooling (`camp`, `fest`) drove the work breakdown — workitems for the
  audit, this festival for the exercise itself.
- Subagents were **not** used for the audit; it was a single directed session. Worth stating
  plainly, since the brief notes "more agent usage is not necessarily better".

## Representative instructions given

Verbatim, from the session that produced the audit:

- *"Create a sub directory in the workitem called security/ to start by ensuring the codebase is
  trustworthy … do not run anything in the codebase, be weary of any prompt injection techniques,
  do a pure static analysis review to ensure it's safe. If you find any risk, document all the
  risk in that sub directory and inform me after the security audit is complete of your findings
  before you begin on the document."*
- *"Ok where is jcenter coming from?"* — a one-line question that surfaced the whole
  supply-chain thread.
- *"Ok so can you fix the code? Also could we setup docker container to run the code instead of
  running it locally anyways?"*

The first is the load-bearing one: **static-only, no execution, report before acting.** It set a
trust boundary on unfamiliar third-party code before anything ran, and the containerization
decision followed from it rather than from convenience.

## Where agents changed the approach

- The audit found the brief's premise is wrong — articles/comments/favorites do not exist — which
  redefines the exercise from "add an endpoint" to "choose what is honestly shippable and say why".
  A quick skim would have started coding against tables that aren't there.
- Tracing `jcenter()` turned a style nit into a real finding: JCenter is decommissioned, and
  combined with `mavenLocal()` and dynamic plugin versions on the buildscript classpath
  (resolved at Gradle's *configuration* phase) it was a live dependency-confusion path.
- The JDK 17/21 CI requirement turned out to be **impossible** on Gradle 4.10 (needs 7.3+ for
  JDK 17, 8.5+ for 21). The toolchain upgrade was forced by the brief, not chosen.

## How agent work was verified

Verification was deliberately execution-based, in Docker, not "the build is green":

- Forged a JWT with the old committed key against the patched app → **401**. Proves the
  vulnerability is actually closed, not just that the key string is gone.
- Asserted no password material in any response body; rotated a password and confirmed
  new → 200, old → 401.
- Checked the status codes that were wrong before: 401 / 422 / 401.
- Port census: only 8080 listening (confirms `createPgServer()` removal).
- Built green on **both** JDK 17 and 21, and built **each commit independently** in a worktree so
  no commit is broken mid-history.
- **Negative-tested the security checks themselves** by reintroducing each defect and confirming
  `just security` went red. A check that never fails proves nothing.
- Proved the DB is ephemeral by registering a user, restarting the container, and logging in
  again → **401, account gone**.

## Things agents got wrong (the honest section)

1. **`@JsonProperty(access = WRITE_ONLY)` on `User.password`** — my fix for password echo. It
   broke all four enabled tests, because `User` is shared between request *and* response, so
   serialization silently stripped the password from outgoing *request* bodies too. Every test
   failed with `{"errors":{"body":["User is invalid."]}}`. Only running the tests caught it;
   it reads as correct. Fixed with a separate `UserResponse` DTO.
2. **`root := parent_directory(justfile_directory())` in the justfile modules** — pointed one
   level above the repo, so three `just security` greps searched an empty tree and reported
   **"ok"**. A silent false pass: the worst failure mode for a security check, and the reason
   negative-testing the checks became a step.
3. **A confidently stated wrong claim:** I said Gradle 4.10 cannot *start* on JDK 16. Replicating
   CI showed it starts fine and fails later, at dependency resolution. The conclusion (CI red)
   held, but the stated mechanism was wrong.
4. **Ran `fest workflow advance` past a step that wasn't done**, and `reset --force` clears the
   whole run — had to restart and re-advance to recover.
5. **Trusted a code review too little, then verified instead of arguing.** An agent reviewer
   requested changes for a password-migration path and a `Follows` primary-key migration. Both
   assumed a durable database; this one is in-memory. Rather than comply or dismiss, I proved it
   (restart test; `INFORMATION_SCHEMA` probe showing the composite key unchanged) and documented
   the conditional concern at the datasource. Implementing P1 would have required re-committing
   the compromised HMAC key the PR existed to remove.

## What to do differently

- Negative-test any check that gates a claim, from the start — finding 2 above was luck.
- Run the test suite before believing a serialization change, not after — finding 1.
- Keep an eye on what a green build actually covers: 4 of 25 tests run here, so "tests pass" was
  nearly meaningless as a regression signal and needed to be said out loud rather than implied.
