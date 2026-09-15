# PLAN Requirements Traceability

Every INGEST requirement (`../../001_INGEST/output_specs/requirements.md`) maps to where the plan delivers it.

| Req | What | Planned in | Decisions |
|---|---|---|---|
| R1 | Ship all three named features | `03_article_search`, `04_popular_articles`, `05_user_activity` | D001 |
| R2 | Tests that prove behavior, edge cases, and no regressions | test tasks `02/06`, `03/05`, `04/04`, `05/03`; the testing gate on every sequence | D004-D010 |
| R3 | CI: build and test, JDK 17 and 21, clear failures, caching | `01/02`, `01/03` | D011 |
| R4 | `AGENT_WORKLOG.md` | `07/02` | D001 |
| R5 | Walkthrough recording | `004_DELIVER` action items | none |
| R6 | Submission reachable by graders | `004_DELIVER` logged-out verification | D012 |
| R7 | Record the departure from "Choose one" | D001; `07/01`, `07/02` | D001 |
| R8 | `unfollow` bug | Deferred, recorded in the work log (`../plan/IMPLEMENTATION_PLAN.md`, Deferred) | none |
| R9 | RealWorld spec tests in CI (bonus) | `06_spec_api_ci` | D011 |
| R10 | CI visible on the fork | `01/01` | D011 |
| R11 | Route base | D002; `07/01` | D002 |
| R12 | Ignored author tests | `02/06`, `04/04`, `05/03` | D010 |
| R13 | New reads are public | `03/04`, `04/03`, `05/02` | D003 |
| R14 | Article foundation | `02_article_foundation` | D008, D009 |
| R15 | Feature semantics | D004, D005, D006, D007 | D004-D007 |

## PLAN outcomes (from `../PHASE_GOAL.md`)

| Outcome | State | Evidence |
|---|---|---|
| Every gap resolved by a decision or left to the user | done | `../inputs/gaps.md` maps each gap to D001-D012; the decision listing and clean cross-reference scan are in `PRESENTATION.md` |
| Implementation plan approved at step 6 | done | judge approval recorded in `PRESENTATION.md`, Approval judge history item 3 |
| Phases scaffolded with executable task files | done | `003_IMPLEMENT` (7 sequences, 27 tasks, 24 gate files) and `004_DELIVER`; the tree is in `PRESENTATION.md` |
| `fest validate` passes with no unfilled markers | done | Score 100/100, 0 markers; complete output in `PRESENTATION.md` |
| User attestation of the plan (PLAN gate step 2) | pending | `operator_attestation`; fest refuses to let a judge decide it, so it needs the user in a terminal |
