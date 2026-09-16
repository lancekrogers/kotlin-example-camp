# Comparator verification outputs

## Match (exit 0)

```
$ docker run --rm -v "$(pwd)/spec-api:/spec" -w /spec python:3.12-slim-bookworm python3 compare_results.py newman-report.json expected-failures.txt
```

Exit code: 0

```
18 failed, 18 expected
```

## Removed one entry (Feed)

```
$ docker run --rm -v "$(pwd)/spec-api:/spec" -w /spec python:3.12-slim-bookworm python3 compare_results.py newman-report.json expected-failures-removed.txt
```

Exit code: 1

```
UNEXPECTED FAILURES (regressions)
  Articles, Favorite, Comments / Feed
18 failed, 17 expected
```

## Bogus entry added

```
$ docker run --rm -v "$(pwd)/spec-api:/spec" -w /spec python:3.12-slim-bookworm python3 compare_results.py newman-report.json expected-failures-bogus.txt
```

Exit code: 1

```
UNEXPECTED PASSES (update the manifest)
  Bogus / Does Not Exist
18 failed, 19 expected
```
