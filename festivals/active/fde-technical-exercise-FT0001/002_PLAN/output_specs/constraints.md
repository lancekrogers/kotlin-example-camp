# PLAN Constraints: how the plan honors each

| Constraint | How the plan honors it | Proof |
|---|---|---|
| C1: only three tables exist | The foundation slice builds only the article, tag-link, favorite, and comment tables the three features read | `../plan/IMPLEMENTATION_PLAN.md` sequences 02, 04, 05; D001 |
| C2: in-memory H2 | Schema is created via `SchemaUtils.create` in repository init, and tests isolate their own data | D009; `02/02` |
| C3: four tests run today | `just test census` output recorded per slice; disabled tests get reasons | D010 |
| C4: toolchain fixed | No Ktor or Exposed upgrade; decisions cite upstream sources at Ktor `1.2.3` and Exposed `0.41.1` | D003, D004, D005 |
| C5: ~90-minute guidance | Ordered slices with a per-slice cutoff | D001 |
| C6: camp commit rules | `fest commit` only, no AI attribution, PR base pinned | D012; `../../gates/implementation/QUALITY_GATE_FEST_COMMIT.md` |
| C7: everything runs in Docker | Tasks run `just build gradle`, `just test all`, `just test jdk 21`, and `just gate`, all containerized | `../plan/IMPLEMENTATION_PLAN.md` overview |
| C8: agent usage is graded | Evidence and missteps captured in each sequence's `results/`; the work log draws from them | D010, `07/02` |
| C9: the festival is replayed | Decisions carry their rejected options; judge histories are recorded in presentations | `../decisions/`, `PRESENTATION.md` |
| C10: ship all three | Delivery order and cutoff | D001 |
