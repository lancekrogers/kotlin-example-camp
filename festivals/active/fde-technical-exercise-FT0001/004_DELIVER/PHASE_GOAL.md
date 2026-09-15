---
fest_type: phase
fest_id: 004_DELIVER
fest_name: DELIVER
fest_parent: fde-technical-exercise-FT0001
fest_order: 4
fest_status: pending
fest_created: 2026-09-15T11:54:21.318145-06:00
fest_phase_type: non_coding_action
fest_tracking: true
---

# Phase Goal: 004_DELIVER

**Phase:** 004_DELIVER | **Status:** Pending | **Type:** Non-Coding Action

## Phase Objective

**Primary Goal:** The submission is complete: the walkthrough is recorded and linked, and both the fork and the recording are reachable by graders.

**Context:** Recording the walkthrough, checking access, and syncing the camp are human or outward-facing steps with no code. They close R5 and R6 once 003_IMPLEMENT has merged.

## Action Items

Tasks to complete during this phase:

- [ ] Record a 5-10 minute walkthrough (screen and voice-over preferred) of what was built and how agents fit the workflow (human work)
- [ ] Add the recording link to `AGENT_WORKLOG.md`, commit with `fest commit`, open a PR against `lancekrogers/kotlin-ktor-realworld-example-app` `master`, and merge it once green
- [ ] Verify from a logged-out browser session that the fork and the recording link are both reachable
- [ ] Run `camp fresh` and `camp refs-sync` so the camp's submodule points at the final fork `master`
- [ ] Record the final state (merged PR list, final CI runs, test census) in this phase's notes

<!-- Add more action items as identified -->

## Prerequisites

What must be in place before starting:

- [ ] 003_IMPLEMENT sequences 01-07 merged, with CI green on the fork's `master`
- [ ] The user has authorized pushes, PRs and merges

<!-- Add more prerequisites as needed -->

## Verification Steps

How to verify each action was completed successfully:

- Recording: plays for 5-10 minutes and shows the three endpoints and the agent workflow
- Link: `AGENT_WORKLOG.md` on the fork's `master` contains the recording link, and it opens for a logged-out viewer
- Access: `curl -sI https://github.com/lancekrogers/kotlin-ktor-realworld-example-app` returns HTTP 200 without credentials
- Sync: `git -C projects/kotlin-ktor-realworld-example-app log -1` in the camp matches the fork's `master`

<!-- Add verification steps for each action -->

## Success Criteria

This action phase is complete when:

- [ ] The fork's `master` contains the implementation, tests, CI, `AGENT_WORKLOG.md` and a working recording link, all reachable logged out
- [ ] All action items verified complete

<!-- Add more success criteria as they become clear -->

## Notes

The recording is human work. Pushes, PRs and merges are outward-facing and need the user's authorization. The PLAN gate step 2 attestation can only be given by the user in a terminal.

---

*Non-coding action phases handle documentation, releases, configuration, and other non-code tasks.*