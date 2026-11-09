---
title: "prwait"
weight: 40
---

# prwait {#prwait}

Wait until a pull request's checks are finished, and **refuse to call an empty check
list a pass** — or a complete, all-green one, when a workflow run for the same commit
is red. [Repository →](https://github.com/go-fleettools/prwait) Latest tag:
**v0.7.0**; this page is written from its README at that tag.

```console
$ prwait -repo go-pkgx/pkgx 82 83
go-pkgx/pkgx#82 PASS SUCCESS:24
go-pkgx/pkgx#83 FAIL test (windows-latest)=FAILURE

$ prwait go-pkgx/bk#311 go-pkgx/mirror#28
go-pkgx/bk#311 FAIL CONFLICTING — rebase it; any green ticks it shows are from the old base
go-pkgx/mirror#28 PASS SUCCESS:20 SKIPPED:1

$ prwait -repo go-tex/go-tex.github.io 239
go-tex/go-tex.github.io#239 FAIL playground (pull_request run)=FAILURE pages (pull_request run)=FAILURE
```

| Exit | Meaning |
| --- | --- |
| 0 | every pull request finished, with checks, and all of them acceptable |
| 1 | one finished red, conflicts, was closed, or ran out of time |
| 2 | refused: bad usage, or an answer that could not be read |

## Install {#install}

```sh
go install github.com/go-fleettools/prwait@latest
```

It shells out to [`gh`](https://cli.github.com), which holds the authentication:
prwait never reads, prints or passes a token.

## Usage {#usage}

```text
prwait [-repo OWNER/REPO] [flags] <pr>...
```

| Target | Asks |
| --- | --- |
| `82` or `OWNER/REPO#82` | a **pull request**: may this land |
| `@main` or `OWNER/REPO@main` | a **branch**: is what landed green |
| `!40237685388` or `OWNER/REPO!402...` | a **workflow run**: did that dispatch finish, and how |

| Flag | Default | |
| --- | --- | --- |
| `-repo` | | repository for bare numbers |
| `-interval` | `45s` | how often to ask |
| `-timeout` | `30m` | give up after this, per run |
| `-min-checks` | `1` | how many checks must **exist** before a pass is possible |
| `-gh` | `gh` | the `gh` binary to ask |

`-min-checks 0` is refused, because it would restore the defect this tool exists
for. The number is one for the whole run, so split the run when suites differ.

## The rule {#rule}

A pull request passes only when **at least `-min-checks` checks exist**, **every one
of them has finished**, and **every conclusion is acceptable**. Absence is never
evidence.

An obvious waiter asks "is anything still pending?" and stops when the answer is no;
that waiter reported success on four pull requests that had never been built. A
conflicted pull request gets no merge ref, so no workflow runs and its check list
stays empty, and the same list is empty right after a push and for the life of a pull
request whose workflow fails to parse. Also: a commit status is not a check run;
`CANCELLED` is not a pass (`NEUTRAL` and `SKIPPED` are); `UNKNOWN` mergeability is not
an answer; a timeout is a failure; an error from `gh` sinks the run.

## And a complete, all-green check list is not a pass either {#runs}

A check run is created **per job**, so a workflow run that fails *before its jobs are
created* leaves no check run at all: nothing red, nothing pending, nothing to count.

Measured on `go-tex/go-tex.github.io#239`, head `769bfee`, 2026-11-09 — v0.5.0 printed
`PASS SUCCESS:3` and exited `0`:

| Asked | Answered |
| --- | --- |
| `commits/769bfee/check-runs` | 3 check runs, all `success`, none pending |
| `actions/runs?head_sha=769bfee` | `docs` ✓, `links` ✓, `pages` **failure**, `playground` **failure** |

`pages` had created **zero** jobs and `playground` one of its three. Every property the
check list can express was true, so from v0.6.0 the pull request and branch forms read
the head commit's workflow runs too and judge them by the same table — `startup_failure`,
the conclusion only a run can carry, is a failure like any other.

Three refinements the same measurement forced:

- **The verdict is bound to the SHA**, never to the branch name or the number: both move
  while a wait is running. An answer carrying no head commit is refused rather than
  judged on its checks alone.
- **The SHA must be whole.** `head_sha=e18cc9b` answers an empty list for a commit that
  has a run in progress; 40 characters finds it. An empty list is a legitimate state, so
  an abbreviation would pass as an absence. Refused before `gh` is reached.
- **Only the latest run of each workflow, per event, is judged** (v0.7.0). A commit
  carries the failed run and the retrigger that replaced it, and judging both leaves a
  commit that is now green unpassable forever. The event is part of that key because on
  that same commit a `workflow_dispatch` was green while the `pull_request` run was red —
  and that dispatch was started by hand to investigate the failure. A run started to look
  into a failure must not be able to clear it.
