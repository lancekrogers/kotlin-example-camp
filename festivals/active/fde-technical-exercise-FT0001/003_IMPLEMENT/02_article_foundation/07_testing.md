---
fest_type: gate
fest_id: 07_testing.md
fest_name: Testing and Verification
fest_parent: 02_article_foundation
fest_order: 7
fest_status: completed
fest_autonomy: medium
fest_gate_id: testing
fest_gate_type: testing
fest_managed: true
fest_created: 2026-09-15T12:06:24.568014-06:00
fest_updated: 2026-09-16T01:46:38.600515-06:00
fest_tracking: true
fest_version: "1.0"
---


# Gate: Testing and Verification

Verify, in Docker, that everything this sequence implemented works, and record the evidence. Run every
command from `projects/kotlin-ktor-realworld-example-app`.

## Commands

- [ ] `just build matrix` ends with `matrix OK: JDK 17 and 21`
- [ ] `just test all` passes
- [ ] `just test census` output is saved to this sequence's `results/`, and no test class is fully disabled without a reason (D010)
- [ ] `just security audit` passes (the supply-chain, secret and wrapper checks from the security review)
- [ ] If this sequence added or changed endpoints: `just docker verify` passes, and each new endpoint was exercised against a running container the way its task describes

## Behavior

- [ ] Every new endpoint has tests for its success path, validation failures (422), not-found (404) where it applies, and anonymous access where it is public (D003)
- [ ] Every endpoint that returns an article or a comment passes `assertNoAuthorSecrets` on the raw JSON (D008)
- [ ] New tests create uniquely named data and assert only on rows they created (D009)
- [ ] Every claim the plan marked unverified that this sequence depends on is now proven by a test or recorded as failing

## Evidence

- [ ] The raw output of each command above (not a summary of it) is saved under `results/`