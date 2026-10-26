---
title: "prwait"
weight: 40
---

# prwait {#prwait}

Wait until a pull request's checks are finished, and **refuse to call an empty check
list a pass**. [Repository →](https://github.com/go-fleettools/prwait) Latest tag:
**v0.5.0**; this page is written from its README at that tag.

```console
$ prwait -repo go-pkgx/pkgx 82 83
go-pkgx/pkgx#82 PASS SUCCESS:24
go-pkgx/pkgx#83 FAIL test (windows-latest)=FAILURE

$ prwait go-pkgx/bk#311 go-pkgx/mirror#28
go-pkgx/bk#311 FAIL CONFLICTING — rebase it; any green ticks it shows are from the old base
go-pkgx/mirror#28 PASS SUCCESS:20 SKIPPED:1
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
