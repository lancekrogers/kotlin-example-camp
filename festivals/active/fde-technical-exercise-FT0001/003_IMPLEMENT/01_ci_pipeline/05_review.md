---
fest_type: gate
fest_id: 05_review.md
fest_name: Code Review
fest_parent: 01_ci_pipeline
fest_order: 5
fest_status: completed
fest_autonomy: low
fest_gate_id: review
fest_gate_type: review
fest_managed: true
fest_created: 2026-09-15T12:06:24.56654-06:00
fest_updated: 2026-09-15T15:01:44.829846-06:00
fest_tracking: true
fest_version: "1.0"
---


# Gate: Code Review

Review every change in this sequence for correctness, fit with the existing code, and the festival's decisions.

## Correctness

- [ ] The code does what the sequence goal and task documents say, including their error paths
- [ ] Validation uses `require(...)`, so failures map to 422, and missing records throw `NotFoundException`, so they map to 404
- [ ] Every handler it wires calls `ctx.respond(...)`, as `UserController` does

## Fit with the codebase

- [ ] It follows the controller → service → repository layering and Kodein wiring already used for users and tags
- [ ] It adds no Ktor, Exposed or Kotlin upgrade (C4), and no new dependency unless a recorded decision covers it
- [ ] Routes stay at root with no `/api` prefix (D002); public reads stay in the optional-auth block registered before the mandatory `authenticate` block, with the comment explaining why (D003)
- [ ] Article and comment authors are `Profile`s; no query reads the password column into a response (D008)
- [ ] No commented-out code, debug output or stray files

## Security

- [ ] No secrets, tokens or real credentials in code, tests, docs or CI
- [ ] Every new CI action is pinned by commit SHA, and nothing new fetches unpinned tools at run time

## Scope

- [ ] The changes match this sequence's goal; stubbed endpoints outside its scope stay stubbed and documented

## Findings

Record every issue that must be fixed before commit in this sequence's `results/`.

**Critical issues:** (must fix)

**Suggestions:** (should consider)