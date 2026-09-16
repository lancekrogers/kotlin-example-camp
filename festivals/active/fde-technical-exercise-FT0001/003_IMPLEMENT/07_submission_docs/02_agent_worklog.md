---
fest_type: task
fest_id: 02_agent_worklog.md
fest_name: agent_worklog
fest_parent: 07_submission_docs
fest_order: 2
fest_status: completed
fest_autonomy: medium
fest_created: 2026-09-15T12:03:15.737203-06:00
fest_updated: 2026-09-16T12:41:43.434454-06:00
fest_tracking: true
---





# Task: Write AGENT_WORKLOG.md

## Objective

Write `AGENT_WORKLOG.md`: short, covering all six points of brief §5, and built from recorded evidence rather than memory.

## Requirements

- [ ] Covers, in this order, the six points the brief lists (`docs/interview-exercise.md:147-152`):
  - the harnesses and models used;
  - a few representative instructions;
  - where agents materially changed the approach;
  - how their work was verified;
  - something they got wrong, missed, or caused a reconsideration;
  - what to do differently next time.
- [ ] Every factual claim cites evidence: a PR, a CI run, a census output, a `results/` file, or a decision record. The log also explains shipping all three features despite "Choose one" (D001), and records the deferred `unfollow` bug (R8)
- [ ] A `## Walkthrough` section holds a placeholder line that `004_DELIVER` replaces with the recording link
- [ ] Written in the submitter's voice as a draft for their review; the brief says "Keep it short. We do not want a transcript." (`:143`), so aim for about 150 lines at most

## Implementation

**Sources.** Read these; don't reconstruct from memory. Festival paths are relative to this festival's own directory. Its location changes as it moves through the lifecycle (`ready/`, then `active/`), so find it with `fest list --all` rather than assuming a path.

- **`001_INGEST/input_specs/agent-usage-record.md`:** the harness, the instructions, and the three mistakes recorded during the audit. Those are the `WRITE_ONLY` serialization break (`:65`), the justfile `root :=` silent false pass, and a wrong Gradle 4.10 claim.
- **`001_INGEST/output_specs/PRESENTATION.md` and `002_PLAN/output_specs/PRESENTATION.md`:** the approval-judge histories.
  - Two rejections were caused by real agent mistakes: an End goal left stale after the scope changed, and a false "no numeric score" claim made from truncated command output.
  - They also record a claim the API contradicted: that GitHub disables Actions on forks.
- **`002_PLAN/decisions/`:** the decisions, and the source reading that changed them. The clearest examples are Ktor's route-order behavior (D003) and the author password-hash leak (D008).
- **Every `003_IMPLEMENT/*/results/` file:** verification evidence and missteps recorded during execution.
- **Merged PRs:** `gh pr list --repo lancekrogers/kotlin-ktor-realworld-example-app --state merged --json number,title,url`.
- **Also include:** planning hit a fest v0.8.0 bug. `fest create phase --dry-run` creates a real phase, which left duplicate phases that had to be found and removed. It is a good example of verifying tool output instead of trusting it.

**Structure**

1. `## Harness and models`
2. `## How I directed the agents`: 3-5 quoted instructions, each with one line on why it mattered. Good candidates:
   - static-only review of untrusted code, reporting before acting;
   - "Shouldn't we use one of the 3 options?", then "Actually let's add all 3";
   - delegating checkpoint approval to an approval judge.
3. `## Where agents changed the approach`: the brief assumed subsystems the code lacks; Ktor route order; the author leak; test data persisting between tests; CI never having run.
4. `## How the work was verified`: for each slice, its PR, CI run and census numbers; the red-path probes; the judge checking planning artifacts against source and the session transcript. State plainly what remained unverified.
5. `## What agents got wrong`: the 3-5 most instructive mistakes, each with how it was caught.
6. `## What I'd do differently`
7. `## Scope decision`: all three features vs "Choose one"; the per-slice cutoff; what stayed stubbed; the deferred `unfollow` bug.
8. `## Walkthrough`: `Recording: (link added after recording, in 004_DELIVER)`

**Checks before handing to the user**

- Tick each of the six §5 points against the brief text (`docs/interview-exercise.md:141`).
- `wc -l AGENT_WORKLOG.md` should be about 150 or fewer.
- Every sentence stating a fact has a link or a path.

**Error paths**

- **A claim has no evidence:** find the evidence, or remove the claim.
- **The file keeps growing:** move detail behind links. Do not paste logs.

## Done When

- [ ] All requirements met
- [ ] `AGENT_WORKLOG.md` covers all six brief §5 points in order, plus the Scope decision and Walkthrough sections; `wc -l` is about 150 or fewer; every factual sentence links to evidence; and the user has reviewed the draft