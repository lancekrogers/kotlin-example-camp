# 004_DELIVER — summary, and what needs a human

## State

The submission is complete and green except for the walkthrough recording.

`master` is `4f751e5`: seven slices merged, 94 running tests with 0 failing and 16 skipped, `just gate` PASSED, CI
green on the last three pushes, both CI gates proven to fail as well as pass, and the fork plus its README and work log
reachable by an anonymous visitor. The camp submodule points at the same commit.

## Needs a human — and cannot be worked around

1. **Record the 5-10 minute walkthrough.** Reserved by the user. No agent-side artifact substitutes for it, and this
   phase's success criteria require a working recording link.
2. **Hand over the link.** Replacing the `## Walkthrough` placeholder in `AGENT_WORKLOG.md` is then one commit, and the
   logged-out check can be re-run against the link itself.

This phase's gate has deliberately **not** been submitted. Its first step asks whether the outcome matches the phase
goal, and the honest answer today is no — one required deliverable does not exist. Submitting would either earn a
correct rejection or, worse, an approval for work that was not done.

## Open recommendation carried from 003_IMPLEMENT

The two deferred test-fidelity fixes were deferred because fixing them meant rewriting published branches that carried
fresh approvals. That reason has expired — everything is merged and those branches are deleted, so the fix is now one
small PR against `master`:

- give `HttpUtil` a `String`-typed raw-JSON helper so `missing body returns 422` exercises an absent field rather than a
  type mismatch;
- make the `%` literal-wildcard assertion discriminating, as the live API check already is.

Neither is a production defect and `master` is green either way. Recommended on the grounds that a submission judged
partly on rigor should not ship a test that cannot fail once the fix is cheap. Not done unilaterally, because
003_IMPLEMENT is closed and approved and reopening it is the user's call.
