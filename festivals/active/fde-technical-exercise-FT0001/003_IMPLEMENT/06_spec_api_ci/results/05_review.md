# Gate 05 results: code review of the spec-test CI slice

**Reviewer:** cursor-agent subagent in read-only ask mode (`composer-2.5`, session
`5fe9a89d-7711-482c-b338-b8eb4b6879f8`, 09:53:43Z to 09:56:20Z). It received the slice-filtered diff, the commit list,
this gate's checklist, the sequence goal, the festival rules, decisions D001/D010/D011 and C7, and every task's
evidence, plus seven specific questions.

**Verdict: REQUEST_CHANGES — no critical defects, but the slice's central claim is unproven.**

The orchestrator agrees with that framing. Everything works locally and the wiring is right, but the one thing this
slice exists to establish — that the comparator actually turns a real GitHub Actions run red — has never been observed
on GitHub. Calling that "done" on local evidence alone would be exactly the kind of unverified claim this festival has
been rejecting all along. It stays open, recorded in this slice's commit gate.

## Findings and dispositions

| # | Finding | Disposition |
|---|---|---|
| S1 | Task 03's red-path proof on a real PR run is missing: break a manifest line, watch the job fail with that request under UNEXPECTED FAILURES, restore it, watch it go green, record both run URLs. | **Accepted, blocked, and tracked.** Cannot be done while this branch sits five deep behind PR #4; pushing would open a PR containing every slice. Listed as outstanding in `07_fest_commit.md` so it cannot be lost. This is the same evidence pattern slice 1 used for the JDK matrix red path. |
| S2 | `compare_results.py` pairs `zip(ordered_names, executions)` with no length check. A collection/report count mismatch would silently mis-attribute results. | **Accepted, gate 06.** The failure mode matters more than its likelihood: a silent mis-attribution makes the gate lie in both directions. It should fail loudly with the two counts before comparing sets. |
| S3 | CI writes `newman-report.json` at the repository root, but `.gitignore:47` only covers `spec-api/newman-report.json`. | **Accepted, gate 06.** A root-level run would leave an untracked 1.3 MB report that could be committed by accident. |
| S4 | The gate is structurally blind to `Delete Comment for Article` and `Delete Article`, which carry zero Postman assertions. | **Accepted as documentation, gate 06.** No code can fix it — the collection has no assertions to observe — but an undocumented blind spot in a gate is worse than a documented one. It goes in the comparator's docstring and `spec-api/README.md`. |
| S5 | CI runs newman on the Actions host via `setup-node`, while `just docker spec` runs it in a digest-pinned container, so supply-chain parity differs. | **Deferred with a reason.** C7 ("Docker only") exists so an unfamiliar third-party toolchain never runs against *this machine*; a disposable GitHub runner is not that machine. Both paths pin `newman@6.2.2` and use the same script, manifest and comparator, so the behavior under test is identical. Recorded as known drift. |

## Checklist (reviewer's result)

| Item | Result | Reason |
|---|---|---|
| Does what the sequence goal and tasks say, including error paths | **fail** | Tasks 01-02 fully evidenced; task 03's PR red-path proof blocked (S1) |
| `require(...)` → 422, `NotFoundException` → 404 | n/a | No application handlers in this slice |
| Every wired handler calls `ctx.respond(...)` | n/a | No Ktor handlers |
| Layering and Kodein wiring | n/a | No application code |
| No Ktor/Exposed/Kotlin upgrade (C4), no new dependency | pass | Shell, Python stdlib and workflow YAML only |
| Routes at root; public reads before mandatory auth | n/a | No routes |
| Authors are `Profile`s; no password in a response | n/a | No responses built here |
| No commented-out code, debug output or stray files | pass | `set -x` gone; no `productionready` reference remains |
| No secrets or credentials | pass | `JWT_SECRET` generated per run; no remote `APIURL` default; `set -x` removed |
| Every new CI action pinned by SHA; nothing fetches unpinned tools | pass, with a noted caveat | 8/8 actions SHA-pinned; `npx --yes newman@6.2.2` is a version-pinned runtime fetch, which D011 chose deliberately |
| Changes match the sequence goal; stubs stay stubbed | pass | Manifest maps only to stubbed endpoints |

## Facts the reviewer verified, worth keeping

- **Manifest integrity, checked against the collection itself.** The official collection contains 31 requests across
  Auth, Articles, Articles/Favorite/Comments, Profiles and Tags — and **no search, popular or profile-stats requests at
  all**. So this festival's three headline features are not exercised by the spec collection; they are covered by the
  94-test suite instead. Every manifest line maps to a list/feed/get/update/comments-list/profile stub, and every
  implemented collection request passes.
- **Gate logic confirmed in both directions**, including `requestError` and `testScript` error handling, and the
  `|| true` on the collection step being what lets the comparator rather than newman decide the job.
- **The pairing's failure mode is mostly safe.** A rename without a manifest update fails loudly. A reorder that
  swapped two requests' names between a passing and failing pair could mis-attribute — unlikely with a checked-in
  static collection, and S2's length check narrows it further.
- **Parity between local and CI** on script, newman version, `APIURL`, manifest and comparator; the drifts are the
  execution environment, the report path, and local's non-zero exit versus CI's `|| true`.
