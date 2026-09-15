---
fest_autonomy: medium
fest_created: 2026-09-15T12:06:24.573866-06:00
fest_gate_id: iterate
fest_gate_type: iterate
fest_id: 06_iterate.md
fest_managed: true
fest_name: Review Results and Iterate
fest_order: 6
fest_parent: 05_user_activity
fest_status: pending
fest_tracking: true
fest_type: gate
fest_version: "1.0"
---

# Gate: Review Results and Iterate

Address every finding from testing and code review, and iterate until the sequence meets its quality standards.

## Findings to Address

### From Testing

- [ ] (list findings from the testing gate)

### From Code Review

- [ ] (list findings from the review gate)

## Iteration

For each finding:

1. Fix the issue
2. Re-run the affected tests with `just test only <Class>`, then `just test all`
3. Record in `results/` what was wrong, how it was caught, and how it was fixed; `AGENT_WORKLOG.md` is written from these records (C8)

## Definition of Done

- [ ] All critical findings are fixed
- [ ] `just test all` passes after the changes
- [ ] Code review findings are addressed or explicitly deferred with a reason
- [ ] Ready to commit