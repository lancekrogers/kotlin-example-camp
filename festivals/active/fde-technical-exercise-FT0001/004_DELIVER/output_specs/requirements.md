# 004_DELIVER — action items, with the output that verifies each

Four of the five action items are verified below. The fifth — the walkthrough recording — is human work reserved by the
user and is the one thing blocking this phase.

---

## Recording a 5-10 minute walkthrough — NOT DONE (human work)

The user reserved this explicitly:

> No I can do the deliver task that doesn't need to be in the festival.

Nothing in the repository or the festival substitutes for it, and no agent-side evidence can be offered. This phase's
success criteria require "a working recording link", so this item and the one below it remain open.

## Adding the recording link to `AGENT_WORKLOG.md` — BLOCKED on the item above

The placeholder is in place and waiting on `master`:

```console
$ grep -n -A2 '^## Walkthrough' AGENT_WORKLOG.md
86:## Walkthrough
87-
88-Recording: (link added after recording, in 004_DELIVER)
```

Once the link exists, replacing that line is a one-commit change.

## Logged-out reachability of the fork and its documents — VERIFIED

Run with `GH_TOKEN` and `GITHUB_TOKEN` unset, so this is genuinely anonymous access rather than an authenticated read:

```console
$ env -u GH_TOKEN -u GITHUB_TOKEN curl -sS -o /dev/null -w '%{http_code}' https://github.com/lancekrogers/kotlin-ktor-realworld-example-app
repo page: HTTP 200
raw AGENT_WORKLOG.md: HTTP 200
raw README.md: HTTP 200
```

The recording link itself cannot be checked until it exists.

## `camp fresh` and `camp refs-sync` so the camp points at the final fork `master` — VERIFIED

```console
$ camp refs-sync
Submodule ref sync plan
  projects/kotlin-ktor-realworld-example-app (up to date)

Skipped submodules:
  projects/kotlin-ktor-realworld-example-app already up to date

All submodule refs are up to date

$ git -C projects/kotlin-ktor-realworld-example-app log -1 --oneline
4f751e5 [amex:bb8421b0-FE-FT0001-PH-003-SQ-07] docs: correct the work log to the final merged state and record the stacked-PR mistake
fork master: 4f751e5
```

The camp submodule pointer and the fork's `master` are the same commit.

## Recording the final state — VERIFIED

**Merged pull requests:**

```console
$ gh pr list --state merged --limit 20
  #11 Integrate slices 3-7 into master: search, popular, user activity, spec CI, docs
  #9  Docs: API additions, CI documentation, and the agent work log
  #8  CI: run the RealWorld collection against a built container, gated by an expected-failures manifest
  #7  User activity: comments and GET /profiles/{username}/stats
  #6  Popular articles: favorites storage and GET /articles/feed/popular
  #5  Article search: public GET /articles/search with validated paging
  #4  Article foundation: create articles with tags, slugs and Profile authors
  #3  CI: JDK 17/21 matrix with pinned actions and JUnit annotations
  #1  Security audit remediation, toolchain modernization, and containerized tooling
```

**Final CI on `master`:**

```console
$ gh run list --branch master --limit 3
  run 35135927840 push success
  run 35135483171 push success
  run 35133183546 push success
```

**Final test census:**

```text
  TOTAL ran=94 passed=94 failed=0 skipped=16

  WARNING: 16 test(s) skipped. A green build does not mean the application works.
```

**Final gate:**

```text
  ok    only port 8080 listening
  ok    token forged with the old committed key rejected (401)
stopped
=== gate: PASSED ===
```

---

## Status

Four action items verified; one not started and one blocked behind it. This phase is **not** complete, and its gate
should not be submitted as though it were. The remaining work is the recording and a one-line edit to
`AGENT_WORKLOG.md` once its link exists.
