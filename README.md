# flight_results

Machine-written. Do not edit by hand.

This repo is a **mailbox**, not a project. An hourly GitHub Actions job in a
separate private repo pulls award availability from the seats.aero API and
drops the raw result here. Nothing in this repo interprets, filters or
decides anything.

It exists because Claude Cowork cloud tasks can read public URLs directly,
but reaching a private repo requires an MCP connector -- and those
[fail to load when a scheduled task fires autonomously](https://github.com/anthropics/claude-code/issues/43397).
A design that reads through a connector works every time you test it by hand
and silently does nothing overnight. Plain HTTPS reads of a public repo avoid
that entirely.

So: the code and the API keys stay private, and only the pulled data is
published here.

## What's here

| File | Contents |
|---|---|
| `data/latest.json` | The most recent pull. Overwritten every run |
| `data/history.jsonl` | One line per run. Append-only |

## Reading it

```
https://raw.githubusercontent.com/japyx/flight_results/main/data/latest.json
https://raw.githubusercontent.com/japyx/flight_results/main/data/history.jsonl
```

## history.jsonl is the health check

Each line looks like:

```json
{"at":"2026-09-13T23:10:09Z","status":"empty","rows":0,"itineraries":0,
 "aa_bookable":0,"first":0,"quota_left":"956","unfiltered_rows":44}
```

`unfiltered_rows` is the one that matters. The pull filters for first and
business class, so a normal quiet day legitimately returns zero rows -- which
is indistinguishable from a broken query unless you check something else.
`unfiltered_rows` is a second, unfiltered probe of the same route and dates.
These routes always carry economy inventory, so:

- `"status":"empty"` with `unfiltered_rows` around 44 means **healthy**, just
  nothing available in a premium cabin.
- `"status":"broken"` with `unfiltered_rows: 0` means the pipeline **is not
  looking**. That is much worse than finding nothing, because it looks
  identical from the outside.

A stale newest `at` timestamp -- more than a few hours old -- means the
GitHub job has stopped running and nobody is watching at all.

## No secrets here

Nothing in this repo is sensitive. Flight availability is public information.
API keys live as encrypted Actions secrets in the private repo; preferences
and Twilio credentials live only in the Cowork workspace. None of them are
published here, and none should ever be added.
