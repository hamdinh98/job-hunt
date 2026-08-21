# job-hunt

Memory for the daily European job hunt routine. The routine runs in an ephemeral
container that is wiped after every run, so anything it needs to remember lives here.

## Files

| File | What it is |
|---|---|
| `european-employers.csv` | The source directory. Which companies are on which ATS, under which slug. |
| `sent-jobs.csv` | Every job ever shown to Hamdi. Checked before sending, appended after. |
| `run-log.md` | Short record of each run — counts, breakage, anything left half-done. |
| `job-hunt-prompt.md` | The prompt the scheduled task runs. Edit here, paste into the schedule. |

## How the directory works

The Greenhouse / Lever / Ashby APIs have no search endpoint. You cannot ask them
"which companies are hiring PHP developers in France" — you can only ask "what is
open at company X", and X has to be spelled exactly right:

```
https://boards-api.greenhouse.io/v1/boards/doctolib/jobs?content=true
                                           ^^^^^^^^ the slug
```

So the bottleneck is knowing the slugs. This file is that knowledge, accumulated.

Every row starts as `candidate` — a guess. Each run resolves 10-15 of them by
actually fetching, and they become `verified` or `dead`. Verified rows are fetched
every run from then on. The list only grows.

## Status values

| Status | Meaning |
|---|---|
| `candidate` | Never successfully tested. board+slug are guesses, not facts. |
| `verified` | Fetched successfully on `last_checked`. Trustworthy. |
| `dead` | Confirmed 404 on all three boards. |
| `no-api` | Real company, but no public JSON board (own careers site, Personio, Teamtailor...). |
| `blocked` | Could not test — network failure. **Carries no information.** Always re-test. |

`blocked` vs `dead` is the distinction that matters most. On 2026-08-21 the routine
could not reach the internet at all; marking those companies `dead` would have
quietly destroyed the list. A network failure is never evidence about a company.

## Current state

Seeded 2026-08-21. **Nothing is verified yet** — the first run was blocked by the
sandbox network policy before it could reach any board. Every row is a candidate
awaiting its first real test.
