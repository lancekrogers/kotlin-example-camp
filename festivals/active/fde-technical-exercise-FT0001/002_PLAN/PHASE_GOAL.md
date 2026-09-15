---
fest_type: phase
fest_id: 002_PLAN
fest_name: PLAN
fest_parent: fde-technical-exercise-FT0001
fest_order: 2
fest_status: completed
fest_created: 2026-09-13T13:15:04.927225-06:00
fest_updated: 2026-09-15T14:32:06.63176-06:00
fest_phase_type: planning
fest_tracking: true
---


# Phase Goal: 002_PLAN

**Phase:** 002_PLAN | **Status:** Pending | **Type:** Planning

## Phase Objective

**Primary Goal:** Plan architecture, design decisions, and task breakdown

**Context:** INGEST established that all three named features read from an article data layer that was never built. Beyond the brief, several open questions affect correctness: the route base, how public reads survive Ktor's route matching, how authors avoid leaking password hashes, and what each count means. Answering them after coding starts would mean rework, so this phase answers them first, turns the requirements into ordered slices, and scaffolds the implementation phases.

## Planning Objectives

- [x] Review the INGEST output specs (step 1)
- [x] Write the gap analysis: `inputs/gaps.md` (step 2)
- [x] Record decisions D001-D012 and index them: `decisions/` (step 4)
- [x] Decompose the work into the festival hierarchy: `plan/STRUCTURE.md` (step 3)
- [x] Write the implementation plan: `plan/IMPLEMENTATION_PLAN.md` (step 5)
- [x] Present the plan and get it approved by the approval judge: `output_specs/PRESENTATION.md` (step 6)
- [x] Scaffold implementation and delivery phases with every marker filled (step 7)
- [x] `fest validate` passes with no unfilled markers (step 8)
- [ ] User attestation of the plan (PLAN gate step 2, `operator_attestation`; needs the user)

## Exploration Topics

What areas need to be explored during this phase:

- How Ktor 1.2.3 picks between matching routes, and what that means for public reads next to `authenticate { }` (D003)
- What Exposed 0.41.1 provides for literal-safe case-insensitive search, grouped counts, and paging on H2 (D004, D005)
- How the untouched domain model would leak user data once filled in (D008)
- How the in-memory database's lifetime affects test isolation and slug collisions (D009)
- Why CI has never run on the fork, and what current, pinned actions look like (D011)

<!-- Add more exploration topics as identified -->

## Key Questions to Answer

Questions that must be answered before this phase is complete:

- In what order do the slices ship, and where is the cutoff? (D001)
- Do new routes live at root or under `/api`? (D002)
- How do Search, Popular and User Activity stay public without changing auth on stubbed routes? (D003)
- What exactly do Search, Popular and User Activity return, including their edge cases? (D004, D005, D006, D007)
- What does an article or comment expose about its author? (D008)
- How are slugs made unique, and how do tests stay independent of each other? (D009)
- Which ignored author tests get enabled, and when? (D010)
- What does CI run, how is it pinned, and why has it never run? (D011)
- How is each slice committed, reviewed and merged? (D012)

<!-- Add more questions as they emerge -->

## Expected Documents

Documents that will be produced during this phase:

- `inputs/gaps.md`: gaps, process blockers, and which decision resolves each
- `decisions/D001`-`D012` and `decisions/INDEX.md`: each decision with its evidence and rejected options
- `plan/STRUCTURE.md`: the festival hierarchy of phases, sequences and tasks
- `plan/IMPLEMENTATION_PLAN.md`: slices, tasks, dependencies, risks
- `output_specs/PRESENTATION.md`, `requirements.md`, `constraints.md`, `context.md`: evidence for the step 6 checkpoint and the PLAN gate

<!-- Add more documents as planning progresses -->

## Success Criteria

This planning phase is complete when:

- [x] Every open question from `inputs/gaps.md` resolved by a decision record, or explicitly left to the user
- [x] The implementation plan is approved at step 6
- [x] Implementation and delivery phases are scaffolded with task files an agent can execute without this conversation
- [x] `fest validate` passes with no unfilled markers

<!-- Add more success criteria as they become clear -->

## Notes

- **Who made the decisions:** the planning agent, under the user's delegation, with the approval judge deciding checkpoints. The user can overrule any decision before execution starts.
- **Blocked on the user:** the INGEST gate step 3 and PLAN gate step 2 attestations. fest refuses to let a judge decide them.
- **Unverified until implementation:** `LOWER` on H2 CLOB columns (D004), H2 GROUP BY rules (D005), rows persisting across test methods (D009), and whether `setup-gradle` validates the wrapper (D011). Each has a named test or check in its task.

---

*Planning phases use freeform structure. Create topic directories as needed.*