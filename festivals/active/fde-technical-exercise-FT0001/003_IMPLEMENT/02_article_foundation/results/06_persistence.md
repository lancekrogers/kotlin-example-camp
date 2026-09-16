# D009 Persistence Probe — Task 06

## Question

Do article rows written by one test method survive into a later test method (new `AppRule` / fresh app, same JVM)?

## Probe

`ArticleCreateTest` with `@FixMethodOrder(MethodSorters.NAME_ASCENDING)`:

1. `a_writes_row` — creates an article with a UUID-suffixed title, stores the returned slug in a companion-object field.
2. `b_row_survives_new_app` — runs immediately after (alphabetically), starts a new `AppRule` instance, then queries `Articles.select { Articles.slug eq storedSlug }.count()` inside a transaction.

## Result

**Rows persist across test methods.**

Both probe tests passed in `just test all` (exit code 0):

```
ArticleCreateTest > a_writes_row PASSED
ArticleCreateTest > b_row_survives_new_app PASSED
```

`b_row_survives_new_app` asserted `count == 1L`, confirming the row written in `a_writes_row` was still present after a new app instance started.

## D009 implication

D009's static reading is **confirmed**: `jdbc:h2:mem:realworld;DB_CLOSE_DELAY=-1` keeps the in-memory database alive for the JVM lifetime, and `AppRule`'s per-method app restart reconnects to the same store. No amendment to D009's stated reason is needed.
