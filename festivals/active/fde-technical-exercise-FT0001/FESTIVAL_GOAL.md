---
fest_type: festival
fest_id: FT0001
fest_name: fde-technical-exercise
fest_status: active
fest_created: 2026-09-13T13:15:04.901304-06:00
fest_updated: 2026-09-15T12:47:04.678052-06:00
fest_tracking: true
---



# fde-technical-exercise

**Status:** Planned | **Created:** 2026-09-13T13:15:04-06:00

## Festival Objective

**Primary Goal:** Ship all three of the brief's named features (Article Search, Popular Articles, User Activity) into the forked kotlin-ktor-realworld-example-app as green, merged slices, with the tests, JDK 17/21 CI, work log, and walkthrough the FDE exercise brief asks for, leaving the fork submission-ready

**Vision:** A grader opens the fork and finds all three named endpoints, each working, tested, and merged through its own green PR. CI runs on JDK 17 and 21 and makes failures readable. `AGENT_WORKLOG.md` and a walkthrough explain what was built and how agents were directed and checked. Every design choice the brief left open is recorded with its evidence and the options it rejected.

## Success Criteria

### Functional Success

- [ ] `GET /articles/search?q=` returns a public, case-insensitive, literal-safe, paged list with the total count
- [ ] `GET /articles/feed/popular` returns a public list ordered by favorite count, then recency, with stable paging
- [ ] `GET /profiles/{username}/stats` returns articles authored, comments written and favorites given
- [ ] Articles, tags, favorites and comments can be created through the existing routes
- [ ] CI builds and tests on JDK 17 and 21 on every PR; the bundled RealWorld collection runs against a live container

<!-- Add more functional outcomes as needed -->

### Quality Success

- [ ] 100% of endpoints returning articles or comments have a raw-JSON test proving no `password`, `email` or `token` under `author`
- [ ] Each of the three endpoints has anonymous-access, validation (422) and not-found (404, where applicable) tests
- [ ] Both JDK 17 and JDK 21 checks are green on every merged PR
- [ ] Every author test left disabled carries a reason string; `just test census` is recorded per slice

<!-- Add more quality criteria as needed -->

## Progress Tracking

### Phase Completion

- [x] 001_INGEST: brief and codebase facts structured into approved specs
- [ ] 002_PLAN: gaps, decisions D001-D012, structure and implementation plan; scaffolded and validated
- [ ] 003_IMPLEMENT: CI, article foundation, the three features, spec-test job, submission docs, each merged green
- [ ] 004_DELIVER: walkthrough recorded and linked; fork and recording verified reachable

<!-- Add phases as they're created -->

## Complete When

- [ ] All phases completed
- [ ] `AGENT_WORKLOG.md` on the fork's `master` covers all six brief §5 points and links the recording
- [ ] The user has given the INGEST and PLAN gate attestations

<!-- Add more completion criteria as needed -->