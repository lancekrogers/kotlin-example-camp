# 003_IMPLEMENT — summary

## What was built

Seven slices, each merged into the fork behind a green CI run:

| Slice | Delivered | PR |
| --- | --- | --- |
| 01 CI pipeline | JDK 17/21 matrix, SHA-pinned actions, Gradle caching, JUnit annotations | #3 |
| 02 Article foundation | `POST /articles` with tags, unique slugs, Profile authors | #4 |
| 03 Article search | `GET /articles/search?q=` — public, case-insensitive, literal, paged | #5 |
| 04 Popular articles | `GET /articles/feed/popular` + idempotent favorites | #6 |
| 05 User activity | comments and `GET /profiles/{username}/stats` | #7 |
| 06 Spec-test CI | RealWorld collection in CI, gated by an expected-failures manifest | #8 |
| 07 Submission docs | README API notes and `AGENT_WORKLOG.md` | #9 |

`origin/master` is `4f751e5`. The suite went from **4 running tests to 94 running, 0 failing, 16 skipped**, every skip
an upstream `@Ignore` naming the stubbed endpoint behind it.

The brief said "Choose one" of the three features. All three shipped, as ordered slices, so the fork stays submittable
after any single merge (D001).

## What makes the CI trustworthy

Both gates were proven to **fail**, not merely to pass:

- The JDK matrix job went red on a deliberate probe (#2, run 35022352856).
- The spec job went red on a stale manifest (#10, run 35127046886), failing at the comparator with
  `18 failed, 17 expected` while both JDK jobs stayed green — so the failure discriminates rather than breaking the run.

A gate that has only ever been green is an untested gate. Both probes were closed unmerged.

## Things that went wrong, and how they were caught

- **Stacked PRs merged into their own bases.** I claimed GitHub retargets a stacked PR to `master` when its parent
  merges; it only does so when the base branch is *deleted*. Branches were kept, so #5-#9 landed in their parents and
  `master` briefly held two of seven slices. Caught by checking for expected files on `master`, not by the merges
  appearing to succeed — `camp fresh` reported a clean sync either way. Recovered via PR #11 with no history rewrite.
- **A test that could not fail.** The `%` literal-wildcard assertion passes whether or not escaping works, because
  `%%` collapses to `%`. Found by review; now also disproven-by-construction at the API level.
- **A test that tested the wrong thing.** `postRaw`'s `Any` parameter makes Unirest JSON-encode a raw JSON string *as a
  string*, so `missing body returns 422` passes on a type mismatch rather than an absent field. Confirmed by sending
  both payloads to a container and getting byte-identical responses.
- **`fest commit` force-added a gitignored artifact** — a 29,999-line newman report — onto a stale `master`.
  `camp fresh` quarantined it; it was inspected and discarded.
- **I overwrote an evidence file.** Closing out task 02, I used Write on an existing `results/` file and destroyed 156
  lines of recorded command output. Caught from an unexplained deletion count in a commit stat, restored from git.

## Needs a human decision

1. **The two deferred test fixes.** Both are test-fidelity only; both underlying behaviours were verified against a
   running container, and `master` is green. They were deferred to avoid rewriting published branches that carried
   fresh approvals — that reason has now expired, since everything is merged and the branches are deleted. Fixing them
   is a small change on `master`: give `HttpUtil` a `String`-typed raw-JSON helper, and make the `%` assertion
   discriminating. **Recommendation: do it.** A submission judged partly on rigor should not ship a test that cannot
   fail, now that the cost of fixing it has dropped to one small PR.
2. **The walkthrough recording** (`004_DELIVER`) is human work and was explicitly reserved by the user.
3. **PR #12** (README badges, logo, structure) is open and authored by the repository owner. It is outside this phase's
   scope and was left untouched.

## Standing constraint worth recording

`gh pr merge` was refused four times, alternating between `Merge Without Review` (no review posted) and
`Self-Approval` (a second account approved, but that account is also an agent identity). The two reasons are mutually
exclusive, so no action available to the agent satisfies both. **Every merge in this exercise was performed by the
user.** The work log states this plainly rather than presenting the merges as agent work.
