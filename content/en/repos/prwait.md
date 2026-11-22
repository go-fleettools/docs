---
title: "prwait"
weight: 40
---

# prwait {#prwait}

Wait until a pull request's checks are finished, and **refuse to call an empty check
list a pass** — or a complete, all-green one, when a workflow run for the same commit
is red. [Repository →](https://github.com/go-fleettools/prwait) Latest tag:
**v0.13.0** (`git ls-remote --tags`, 2026-11-22); this page is written from its
README at **v0.13.0**, and the two are stated separately on purpose: *which tag is
current* and *which tag this page describes* are different facts.

```console
$ prwait -repo go-pkgx/pkgx 82 83
go-pkgx/pkgx#82 @4f1c2ab PASS SUCCESS:24
go-pkgx/pkgx#83 @9d0e7f1 FAIL test (windows-latest)=FAILURE

$ prwait go-pkgx/bk#311 go-pkgx/mirror#28
go-pkgx/bk#311 FAIL CONFLICTING (#311) — rebase it; any green ticks it shows are from the old base
go-pkgx/mirror#28 @1b77c04 PASS SUCCESS:20 SKIPPED:1

$ prwait -repo go-tex/go-tex.github.io 239
go-tex/go-tex.github.io#239 @769bfee FAIL playground (pull_request run)=FAILURE pages (pull_request run)=FAILURE
```

| Exit | Meaning |
| --- | --- |
| 0 | every pull request finished, with checks, and all of them acceptable |
| 1 | one finished red, conflicts, was closed, moved, or ran out of time |
| 2 | refused: bad usage, or an answer that could not be read |
| 3 | the API quota stopped the wait: nothing was measured, nothing is claimed |

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

## The verdict names the commit it judged {#named}

Every verdict carries the head it was decided at (`@4f1c2ab` above), because a wait that
began on one commit and a head it ended with are different pieces of work. A target with
no commit to name — a conflicted pull request has no head to judge — prints none rather
than an empty `@`.

⛔ **And a head that moves mid-wait is refused, not followed** (v0.9.0). Naming the commit
is necessary and not sufficient: a wait that silently re-points at a push the caller never
asked about ends by printing a true statement about the wrong work.

```console
$ prwait -repo go-tex/engine 752
go-tex/engine#752 MOVED @e87aa0a -> @2baff07 — the head changed while waiting; this wait judged neither, re-run against @2baff07
```

Neither commit was judged as a whole, so neither is reported as a verdict, and the exit
status is `1`. It holds for a branch (`@main`) and a run as much as for a pull request — a
release gate waiting on the default branch must not answer about a commit that arrived
after the one it was asked about, or the tag lands on something nobody judged. A SHA that
one poll could not read is **not** a move: a blank answer is weather, and the retry loop
already covers it.

## A `@branch` whose head is a conflicted pull request {#branch-conflict}

GitHub runs `pull_request` workflows against `refs/pull/N/merge`, which does not exist
while the merge cannot be computed — so a conflicted pull request has **no runs at all**,
by construction. `prwait` has refused `CONFLICTING` before counting checks since its first
release, but the branch form supplied a fabricated `mergeable: "MERGEABLE"` — right for a
bare branch, which has nothing to merge into, and a lie when the branch is a pull
request's head. The refusal was therefore unreachable from the form that avoids guessing a
number, which is the form the fleet's own rule prescribes (fixed in v0.11.0).

Measured 2026-11-18 on `go-zpkg/zpkb#505`: `TIMEOUT after 30m0s — 0 checks, want at least
1`, then three retriggers spent on a dropped-dispatch theory — Actions enabled, all
workflows `active`, repository public, githubstatus `operational` — while `gh pr view
--json mergeable` answered `CONFLICTING` in one call. The refusal now names the pull
request, because a reader told "CONFLICTING" about a *branch* has no way to know which one
said so.

⚠ The selection is by **head**, not by association: GitHub associates a commit with every
pull request that contains it, so a branch cut from `main` is associated with whatever
merged into `main`. Only a pull request whose `headRefName` is this branch says anything
about what this branch would merge.

## A red that can no longer be re-run says so {#rerun}

Past GitHub's 30-day re-run window, `gh run rerun` answers `403 Unable to retry this
workflow run because it was created over a month ago` — so the red is permanent where it
stands, and the only way to a fresh verdict is a ref you control (v0.12.0).

```console
$ prwait -repo go-ansible/playbook 197
go-ansible/playbook#197 @c5dd86d FAIL test (macos-latest)=FAILURE
  ⛔ ci (pull_request run #412) was created 2026-10-12, 37 days ago: `gh run rerun` will answer 403 (GitHub allows a re-run for 30 days).
     This red cannot be refreshed where it stands. Reproduce the lane locally if it names your platform, or put the commit on a ref you control — `git cherry-pick` onto a branch in this repository — and let CI judge it there.
```

That line exists because the question went unasked for ten days on that very pull request.
It was handed back twice as *"blocked: three routes tried, all closed"* — and all three
routes were ways to make **that** run go green, so they shared one premise and tested it
once. Cherry-picked onto a branch in the target repository, the same commit came back
**9/9 green, `macos-latest` included**: the red belonged to the stale base, not to the
change.

## And a rate limit is not a verdict {#quota}

When GitHub refuses the call for the quota, nothing about the target was measured, so
nothing is claimed about it (v0.13.0):

```console
$ prwait -repo go-embedded-julia/julia 78 79 80 82
go-embedded-julia/julia#78 QUOTA — not a verdict: nothing about this target was measured.
  ⛔ the per-user REST quota is spent (0 of 5000 left). It resets at 20:45 CET, in 38m12s.
     The quota is per USER, not per token, so every session on this machine shares it — Renovate included.
⛔ 4 target(s) got no verdict at all: …#78, …#79, …#80, …#82 — the quota stopped the wait before they were read.
```

Measured 2026-11-15: the account's REST quota hit zero twice in one day, and both times
this tool printed the raw `gh: API rate limit exceeded … (HTTP 403)` *where its verdict
goes*, under **exit 1 — the status of a red check**. A caller reading only the status took
"nothing could be measured" for "the pull request is red", which is this tool's own defect
wearing its least recognisable face: an **unread** answer presenting itself as a
**negative** one.

The retry made it worse. A failed query is weather here, so a quota was asked again three
times at the poll interval — three more refusals, each spending a call against the very
limit that was refusing. **Weather has no reset time; a quota has one, advertised.** So a
quota now gets its own classification, its own hold, and its own exit status:

- **Held for, when the reset fits.** The wait sleeps to the advertised reset and resumes.
  Costs nothing, and keeps the verdict.
- **Refused early, when it does not.** Spending the rest of the timeout to learn what
  `/rate_limit` already said is how four calls bought one fact.
- **A secondary limit is reported as a secondary limit.** `/rate_limit` cannot see it —
  *"There is not a way to check the status of your secondary rate limit"* — so when GitHub
  refuses for a rate limit while the instrument reports room, that is said in those words
  and marked as **deduced**. No reset is ever borrowed from the primary limit to stand in
  for one that does not exist.
- **A measured red still outranks it.** If anything was actually read as failing, the
  status stays `1`.

⚠ `GET /rate_limit` is free of the *primary* limit only — *"Calling this endpoint does not
count against your primary rate limit, but it can count against your secondary rate
limit"* — so it is read once before the wait and once per refusal, never per poll.
