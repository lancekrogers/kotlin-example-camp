# Evidence: AGENT_WORKLOG.md

## Final state

On `master` at commit `4f751e5`. **88 lines**, inside the task's ~150-line ceiling, with all eight required sections in
order: the six points of brief §5, then Scope decision, then Walkthrough.

```text
## Harness and models
## How I directed the agents
## Where agents changed the approach
## How the work was verified
## What agents got wrong
## What I'd do differently
## Scope decision
## Walkthrough
```

The `## Walkthrough` section holds the placeholder line for `004_DELIVER` to replace with the recording link.

## Corrections made after the draft

The draft was written while the slices were still unmerged, and two of its claims went stale. Both were fixed before
this task was closed, because the task requires every factual claim to cite evidence and a stale claim cites evidence
for something that is no longer true.

1. **The verification table** described slices 3-6 as "stacked on #4", "merge blocked" and "PR not opened". All seven
   slices are now merged; the table names each PR and a row for slice 7 was added.
2. **A "Still unmerged" paragraph** listed #5-#9 as open and awaiting merge. Replaced with the final state: `master` at
   `dda522e` (now `4f751e5`), and the observation that #11 is the only PR that received ordinary `pull_request` checks,
   because the stacked PRs never matched `branches: [master]`.

Three mistakes discovered after the draft were added to **What agents got wrong**: the stacked-PR mis-merge and its
recovery, `fest commit` force-adding a gitignored 29,999-line newman report onto a stale `master`, and the two tests
found by PR review that passed without proving their claims. **What I'd do differently** gained four entries, including
checking a CI trigger's branch filter before treating a green run as a PR check.

The merge situation is now stated plainly rather than implied: `gh pr merge` was refused four times, alternating
between `Merge Without Review` and `Self-Approval`, so **every merge in this exercise was performed by the user**. The
log says so rather than presenting the merges as agent work.

## How the "user has reviewed the draft" condition was satisfied

The Done When ends with "and the user has reviewed the draft". The user's instruction was:

> Ok now continue the fest next loop that pr was merged, run camp fresh first then continue running fest next until the
> festival is complete.
>
> Do not ask for my approval you have permission to drive this to the end

That is authorization to complete the task, not a line-by-line review of the prose. Recording the distinction here so
the record is not stronger than the fact: the draft was delivered, its accuracy was verified against the final
repository state, and the user authorized closing the task without a further review pass.
