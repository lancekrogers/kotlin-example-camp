---
fest_type: phase
fest_id: 001_INGEST
fest_name: INGEST
fest_parent: fde-technical-exercise-FT0001
fest_order: 1
fest_status: completed
fest_created: 2026-09-13T13:15:04.905901-06:00
fest_updated: 2026-09-15T14:31:22.758357-06:00
fest_phase_type: ingest
fest_tracking: true
---


# Phase Goal: 001_INGEST

**Phase:** 001_INGEST | **Status:** Pending | **Type:** Ingest

## Phase Objective

**Primary Goal:** Ingest and structure input materials into actionable specifications

**Context:** The FDE exercise brief was seeded at creation and renamed to `input_specs/exercise-brief.md`. Two further inputs were gathered from this camp's security audit and re-verified against the merged `master` (`bf1435e`). All three are transformed into the structured output specs below for planning.

## Input Sources

Place all raw input materials in `input_specs/`:

- [x] `input_specs/exercise-brief.md` — the FDE exercise brief, a verbatim transcription of `docs/interview-exercise.pdf` (seeded; byte-identical to `docs/interview-exercise.md`)
- [x] `input_specs/codebase-state-verified.md` — codebase facts from the security audit re-verified on `bf1435e`, plus feature-sizing and route-base evidence gathered at step 5 (agent-written, with `file:line` and commit anchors)
- [x] `input_specs/agent-usage-record.md` — raw material for `AGENT_WORKLOG.md` captured from the audit sessions (agent-written; it exists nowhere else)

## Expected Outputs

The following structured documents will be created in `output_specs/`:

| Output | Purpose |
|--------|---------|
| `purpose.md` | Festival purpose, success criteria, motivation |
| `requirements.md` | Prioritized requirements (P0/P1/P2) with traceability |
| `constraints.md` | Technical and process constraints |
| `context.md` | Prior art, related systems, key references |

## Success Criteria

This ingest phase is complete when:

- [x] All input sources reviewed and understood
- [x] Output specs created following standard structure
- [ ] User has approved the structured output — the user reviewed the step 5 summary, gave two corrections (both incorporated), and delegated the approval decision to the approval judge rather than approving directly
- [ ] No unresolved questions or ambiguities — open items are carried into PLAN as decisions; see Notes

## Workflow

This phase uses step-based workflow guidance. See `WORKFLOW.md` for the step-by-step process.

Use `fest next` to see the current step.
Use `fest workflow advance` to move to the next step.

## Notes

**User direction incorporated at the step 5 checkpoint:**
1. Ship one of the brief's three named options, not a substitute.
2. Revised: ship all three named features in this festival (C10, R1, R7).

**Carried into 002_PLAN as decisions, not resolved here:**
- Route base: root (recommended, with evidence) vs `/api` — R11.
- Per-feature semantics: Search's match rule and `q` handling, Popular's tie-break, what User Activity's `favoritesCount` counts, and what `articlesCount` means — R15.
- Auth placement for the three public read endpoints — R13.

**Static claims not yet proven by execution:**
- `unfollow` deletes the wrong row orientation (R8, P2).
- Ktor 1.2.3 routes constant segments (`search`, `feed`) ahead of `{slug}`.
- What Exposed 0.41.1 offers on H2 for case-insensitive LIKE and escape characters.

**Structure:** festival-level markers (`FESTIVAL_GOAL.md`, `FESTIVAL_OVERVIEW.md`, `TODO.md`) and `002_PLAN/PHASE_GOAL.md` markers are still unfilled. `fest validate` reports Score 90/100: structure, completeness, task files, quality gates, ordering, auto-link, hooks, and workflow pass, with 27 unfilled markers in 4 files.

**INGEST completion caveats (step 6):**
- Step 5 was approved by the approval judge (`judge-agent`, claude preset), not by the user. The user delegated that decision and has not attested the specs themselves.
- Judge history, five runs: a readiness block (missing `PRESENTATION.md`), a parse failure (verbose CLI output), two rejections on real defects (a stale End goal, and a false "no numeric score" claim), then approval. Details are in `output_specs/PRESENTATION.md`.
- All INGEST work is uncommitted in the camp root, including `festivals/.festival/config.yaml` and `festivals/.festival/judge-agent.json`.

---

*Ingest phases transform unstructured input into structured specifications ready for planning.*