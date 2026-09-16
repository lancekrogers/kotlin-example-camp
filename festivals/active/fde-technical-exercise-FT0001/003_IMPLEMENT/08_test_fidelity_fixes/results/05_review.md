# Gate: Code Review

Reviewed by an independent read-only reviewer against the working-tree diff. **Verdict: APPROVE**, no critical issues.

## Confirmed by the reviewer

- **No production code changed.** The diff is exactly four files under `src/test/` plus `AGENT_WORKLOG.md`. The
  reviewer re-read `ArticleRepository.kt:100-101` separately and confirmed `search()` and the `LikePattern`
  construction are untouched.
- **`postRawJson` typing, verified against the library rather than the comment.** The reviewer checked Unirest 1.4.9
  itself: `body(String)` and `body(Object)` are genuinely distinct overloads, and `RequestBodyEntity.body(Object)`
  looks up the registered `ObjectMapper` and calls `writeValue(body)`, while `body(String)` assigns the string with no
  serialization. Since overload resolution is static, an `Any` parameter holding a String still binds `body(Object)`.
  Their words: a real, version-verified footgun rather than a folk claim.
- **The guard test discriminates**, and `missing body returns 422` structurally cannot — `{"comment":{}}` is invalid
  under both readings, so both produce 422. That is exactly the gap the original defect hid in.
- **The `%` decoy is a real decoy**, traced through the pattern: escaping intact requires the literal substring
  `…100%`, which the decoy lacks; escaping removed leaves `%%`, which collapses to one wildcard and matches anything
  containing `…100`, decoy included.
- **Nothing weakened.** Four removed lines, all renames or a call-site swap — a variable rename and `postRaw` →
  `postRawJson`. No assertion deleted or loosened, underscore assertions untouched, no test removed or newly `@Ignore`d.
- **Hygiene clean.** No debug output, `TEMPORARY` markers or commented-out code; no secrets. Test data is
  `UUID.randomUUID()`-scoped into slugs, titles and usernames, so nothing depends on row counts in the shared
  JVM-lifetime H2 database. The reviewer additionally confirmed the decoy's slug and title cannot collide with any
  other test's data.

## Findings

**Critical issues:** none.

**Suggestions:**

1. **`json` parameter shadows a class field.** `HttpUtil.kt:51`'s parameter is named `json`, and the class has
   `private val json = "application/json"` at `:13`. It resolves correctly and the body does not need the constant, so
   this is readability rather than a bug. **Addressed** in the iterate gate: renamed to `rawJson`.

## Process note from the reviewer, and why it is expected

The reviewer observed that `master` and `fix/test-fidelity` point at the same commit, so
`git diff master...fix/test-fidelity` is empty and they reviewed the unstaged working-tree diff instead.

That is correct and is how this festival's gates are ordered: the review gate (05) runs before the commit gate (07), so
at review time the work is deliberately uncommitted. The reviewer read the right content. Their recommendation to
commit before the branch is diffable is satisfied by gate 07, which commits with `fest commit` and opens the PR.

## What the reviewer explicitly did not verify

Build and test execution, as instructed. `AGENT_WORKLOG.md`'s "95 running, 0 failing, 16 skipped" and the red/green
runs rest on the orchestrator's own recorded output in `results/03_verify_both_directions.md` and `results/04_testing.md`,
not on anything the reviewer executed. Stated here rather than glossed.
