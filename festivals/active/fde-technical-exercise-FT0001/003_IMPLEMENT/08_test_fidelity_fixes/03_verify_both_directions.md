---
fest_type: task
fest_id: 03_verify_both_directions.md
fest_name: verify both directions
fest_parent: 08_test_fidelity_fixes
fest_order: 3
fest_status: pending
fest_autonomy: medium
fest_created: 2026-09-16T12:59:38.412761-06:00
fest_tracking: true
---

# Task: Prove each changed test fails on broken behaviour before proving it passes

## Objective

Show, with recorded output, that each of the two rewritten tests fails when the behaviour it checks is deliberately
broken, and passes against the real code. Then update `AGENT_WORKLOG.md` so the deferred list matches reality.

## Requirements

- [ ] For the `%` test: escaping is deliberately removed, both rewritten tests fail, the failure output is recorded,
  and the change is reverted
- [ ] For the missing-field test: the helper is deliberately pointed back at the object-serializing overload, the test's
  wire payload is shown to change, and the change is reverted
- [ ] Each failing run is followed by a recorded passing run against the restored code
- [ ] `AGENT_WORKLOG.md`'s "Deferred test fixes" section is rewritten to say these two are now fixed, keeping the
  deferred `unfollow` bug (R8) entry
- [ ] `git status` is clean of the deliberate breakages before the sequence's commit gate

## Implementation

This is the task the sequence exists for. A test that has only ever been observed green is exactly the defect being
removed, so both directions must be recorded or the sequence has replaced two unfalsifiable tests with two more.

**Steps**

1. **Red for `%`.** In `src/main/kotlin/io/realworld/app/domain/repository/ArticleRepository.kt`, temporarily replace
   the escaped pattern with an unescaped one:
   ```kotlin
   // TEMPORARY, for red-path proof only. Revert.
   val pattern = LikePattern("%" + term.lowercase() + "%", '\\')
   ```
   Then run only the two affected classes in Docker:
   ```bash
   just test only ArticleSearchRepositoryTest
   just test only ArticleSearchTest
   ```
   Expect `percent sign in search term is literal` to fail — either on `page.total` being 2 or on `.single()` throwing
   — and the HTTP percent assertion to fail on `articlesCount`. Paste the failure text into
   `results/03_percent_red.txt`. Check the exact recipe name with `just --list` first; if `just test only` takes a
   different argument shape, use whatever the module actually exposes and record the command used.
2. **Revert and go green.** `git checkout -- src/main/kotlin/io/realworld/app/domain/repository/ArticleRepository.kt`,
   re-run both classes, and paste the passing output into `results/03_percent_green.txt`. Confirm with
   `git status --short` that the file is unmodified.
3. **Red for the missing-field test.** Temporarily change `postRawJson`'s parameter from `String` to `Any` in
   `HttpUtil.kt` — that alone reproduces the original defect, since the overload is chosen from the declared type.
   Capture the wire payload the same way task 01 did (container `curl` comparison, or a temporary `println` of the
   serialized body). Record in `results/03_rawjson_red.txt` that the body became `"{\"comment\":{}}"` while the status
   stayed 422. **The status will not change**, which is the point: record that the response is identical and that only
   the payload differs. If a `println` was used, note its removal.
4. **Revert and go green.** Restore the `String` parameter, re-run `just test only CommentCreateTest`, and record both
   the passing result and the wire payload `{"comment":{}}` in `results/03_rawjson_green.txt`.
5. **Full suite and gate.** Run `just test census` and `just gate`, recording both. `ran` must be 94 or higher with
   `failed=0` and `skipped=16`; the gate must end `=== gate: PASSED ===`.
6. **Update `AGENT_WORKLOG.md`.** Its "Deferred test fixes" section currently says both items are deferred, with the
   justification that fixing them would rewrite published branches carrying fresh approvals. That justification expired
   when everything merged and those branches were deleted. Replace the section with a short entry saying both were
   fixed in this sequence and how each is now falsifiable, and keep the R8 `unfollow` entry, which remains genuinely
   deferred. Keep the file at or under the ~150-line ceiling the slice-07 task set.
7. **Confirm nothing temporary survived.**
   ```bash
   git status --short
   grep -rn 'TEMPORARY\|println(' src/main src/test --include='*.kt'
   ```
   Both must come back clean of anything this task introduced. Quote the `--include` glob; an unquoted `*.kt` is
   expanded by zsh before grep sees it.

**Error paths**

- **A test still passes with the behaviour broken:** the rewrite did not discriminate. Do not record it as proven.
  Return to task 01 or 02 and strengthen the assertion, then repeat this task.
- **`just test only <Class>` is not a real recipe:** read `.justfiles/test.just` and use the recipe that exists; never
  fall back to running Gradle on the host (C7).
- **The deliberate breakage cannot be reverted cleanly:** `git diff` before reverting, and restore from `origin/master`
  if needed. Never commit a breakage, even temporarily.

## Done When

- [ ] All requirements met
- [ ] `results/` holds four files showing red then green for each of the two tests, plus a census at `ran>=94 failed=0
  skipped=16` and a `=== gate: PASSED ===`; `AGENT_WORKLOG.md` no longer lists these two as deferred while still
  recording R8; and `git status --short` shows no deliberate breakage left behind
