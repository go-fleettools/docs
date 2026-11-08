---
title: "fleettools"
weight: 10
---

# fleettools {#fleettools}

Sweeps across about 1800 repositories and about 320 organisations: which default
branches are red, which runners stopped or run and fail, which modules are ahead of
their tag, which foreign judges the CI never installs, and which tests exist without
ever running. [Repository →](https://github.com/go-fleettools/fleettools) Latest tag:
**v0.84.0** (`06dfd81`, 2026-11-07, measured the same day). The prose below was
written from the repository at **v0.67.0**, plus the two v0.82.x sections at the
end written from the change itself — the command inventory under `cmd/`,
the README's own sections, and each new command's doc comment — and the two are kept
apart on purpose: *which tag is current* and *which tag this page describes* are
different facts, and writing one where the other belongs is how a page comes to
assert a version it was never checked against. The README is about 610 lines and
describes every command in its own words; read it there.

The sweeps were written on 2026-08-30 while getting every default branch green, and
the README says why each one is built the way it is. The repository has **one
directory per sweep under `cmd/`**; the module is
`github.com/go-fleettools/fleettools`, and a command is installed the way Go installs
tools, at a tag (`go install github.com/go-fleettools/fleettools/cmd/vulngate@v0.81.2`).

## The commands {#commands}

**65 directories under `cmd/`**, each with a `main.go` (counted 2026-11-07 with
`go list ./cmd/...` on `06dfd81`, the commit that merged the 65th; it was 64 at
v0.81.2, counted the same day on `4bc11f15`):

```text
abbench blobpurge branchscan brandcheck brief buildscan buildtag bzcomment clonescan
covblocks cronscan docscan docscheck docsversions docsym flagcensus fleetverbs ghissue
glmerge gofloor gopin gorequires gvlscan hostload i18ncheck judgescan landingmigrate
langcheck linkcheck mantracheck mdpdf mkdocs2hextra modrename needread pagesbuild
pagesbuildadd pagesbuildcampaign pagescheck pagespublish pagesretire pagetext pinscan
pngeq priorart prmerge prrefresh prscan qemufix quietscan redscan relverify sitelinks
sitemigrate symfind tagcells tagscan testscan treecmp truncspill unarc vscan vulngate
vulnscan wasmsize wfaudit
```

The six added since v0.78.0 are `fleetverbs`, `ghissue`, `pagesbuild`,
`pagesbuildadd`, `pagesbuildcampaign` and `priorart`; the four added before them,
since v0.67.0, were `branchscan`, `bzcomment`, `linkcheck` and `prrefresh`.

⛔ **This count has now gone stale twice**, which is worth keeping on the page
rather than quietly correcting. It said **55** for four days, then **59** against
a repository that held 64 — and each time nothing recounted it, because a number
on a page is something a reader has no way to doubt. The remedy that works is the
one used here: cite the **commit** the count was taken on, so the claim carries
its own expiry. `docsversions` is the tool for the version a page claims, and
`docscan -fail-on-drift` reported **0 drift** throughout both episodes — correctly
by its own definition, and blind to a version asserted in prose.

Grouped by the question they answer:

- **Is the fleet green?** `redscan` (every default branch whose last run failed — a red
  branch is not a broken one, check *when* it last ran), `quietscan`, `prscan`,
  `prmerge`, `glmerge` (the GitLab side), `relverify`, `cronscan`.
- **Can this pull request still go green?** `prrefresh` (updates a pull request branch
  with its base and names the ones it **could not** — a pull request in conflict with
  its base gets *no workflow run at all*, not a failing one, so it keeps its last green
  and reads as ready). It sends `expected_head_sha`, so it refuses rather than merges
  into a head that moved under it, and it treats GitHub's `mergeable: null` as *"the
  background job has not finished"* rather than as a refusal — reading `null` as `false`
  reports a conflict for every pull request asked about too soon.
- **Is what is merged reachable, and is the version right?** `tagscan` (a module whose
  branch is ahead of its latest tag: work that is merged and `go get` cannot reach — it
  asks about **tags**, not releases), `vscan` (what the next version should be, derived
  from the public API with `gorelease`, once per GOOS), `tagcells`, `gorequires`,
  `pinscan`, `modrename`.
- **Is there work nobody can find?** `branchscan` (branches holding commits the default
  branch does not have that **no pull request was ever opened for**). Such a branch is
  invisible to `gh pr list` and to any backlog that tracks issues, so it does not read
  as abandoned — it reads as absent, and the work gets done a second time. The cousin of
  `tagscan`: that one finds work merged and unreachable, this one finds work nobody
  ever saw. `priorart` asks the same question from the other end, about a **subject**
  rather than a population, and *before* the work starts: given a repository and a
  subject it reads the branches — by name **and** head-commit subject, since a branch
  is named for its mechanism and its commit written for its cause — and the issues and
  pull requests, **open and closed**, and exits non-zero when the subject is already
  covered. No `is:open` qualifier, deliberately: a closed "not merged, and here is
  why" is the most valuable prior art there is. It exists because three pieces of work
  were redone on one repository in a single day while "search the tracker first" was
  already written down as a rule — the thought arrives as *let me fix this*, never as
  *let me check whether this is fixed*, so the check runs instead of being remembered.
  A surface it could not read holds the gate shut (status 2), including a GraphQL
  refusal, which arrives as HTTP 200 with an `errors` array: read past that and a
  refused query becomes an empty branch list, which is a clean gate on a surface never
  read. It keeps the search's own ranking, because a scoring pass of its own was
  measured against it and came out worse.
- **Does the CI actually test it?** `testscan`, `judgescan` (tests that look for an
  external tool the CI never installs — a foreign judge that exists in the source and
  has never once run), `covblocks` (the blocks a coverage profile never entered, because
  `go tool cover -func` rounds to one decimal and a 100 % gate passes at 99.95 %),
  `buildscan`, `buildtag`, `flagcensus`.
- **Is it safe?** `vulnscan`, `vulngate` (the per-repository half), `gofloor` (which Go
  toolchain a repository can be built with, and whether CI would notice a vulnerable
  one), `wfaudit`, `blobpurge`, `needread` (the `DT_NEEDED` entries of every ELF under a
  tree, refusing to report unless a symbol it was told to expect is actually there),
  `symfind` (whether an ELF *exports* a dynamic symbol, rather than merely consuming it).
- **Is it advertised, and is the page what it builds?** `docscan`, `docscheck` (which
  generator a published URL was built with), `pagescheck` (whether a Pages site serves
  what its repository builds), `pagetext` (whether a published page now carries the text
  we wrote), `sitelinks` (whether every local link of a built site leads to a file of
  that build), `linkcheck` (whether every **external** link a documentation source names
  still resolves — it replaced the `grep`/`sed`/`curl` the documentation workflows
  carried, which failed twice on random 504s from github.com and passed on the third
  run), `docsym`, `i18ncheck`, `langcheck`, `mantracheck`, `brandcheck`.
- **Speaking to something that is not GitHub.** `bzcomment` posts one comment to a
  Bugzilla bug through its REST API, showing which bug and which account it is about to
  speak for first. It exists so a session can contribute upstream without the API key
  ever passing through a URL, an argument vector, the environment or a log.
- **Publishing the documentation.** `pagespublish`, `pagesretire`, `mkdocs2hextra`,
  `sitemigrate`, `landingmigrate`, `docsversions` (every published version of a
  versioned site built with **one** look — the default branch's), `mdpdf`.
- **What does it cost, and on what?** `wasmsize`, `hostload`, `abbench`, `truncspill`,
  `gvlscan`, `gopin`, `qemufix`. Since v0.71.0 `abbench` also runs on macOS and can
  compare two sub-benchmarks of one binary: `--max-load F --wait D` waits for the
  1-minute load to be at most F before each round and **discards a round's pair** when
  the load after it is above F, so the two series stay paired; `--base-arg` and
  `--head-arg` add arguments to one side, and `--rename-base` / `--rename-head` give
  the two sub-benchmarks one name so that `benchstat base.txt head.txt` compares them
  (benchstat keys rows by full name). The load reading is the public package
  `github.com/go-fleettools/fleettools/loadavg` (`Avg1`, `Wait`), also used by
  `go-llmflow/bench`'s `gwbench`.
- **Comparing artefacts.** `treecmp` (byte for byte), `pngeq` (the same *pixels*, because
  "65 of 73 files differ" on a moved brand kit was false — 51 PNGs, 0 channel values
  different).
- **The small ones.** `brief` (runs a command, shows the end of what it said, and keeps
  its exit status), `unarc`, `truncspill`, `clonescan`.

## What is built into them {#built-in}

- **They say when they are not the code in this repository.** A binary installed from
  an old tag is compared with what has changed for that command since.
- **Every one answers `-h` with 0.** `redscan -h` once ran a full sweep of every
  organisation and had to be killed; a test now fails any command that cannot answer
  `-h`.
- **A sweep asks for the hourly API budget before spending it** and refuses to begin
  below a floor, because that budget is **per user** and shared with the fleet's
  Renovate runners. Four sweeps have emptied it and made those runs fail.
- **A failed call must never become a verdict**, and a deliberate budget stop is
  reported separately from "could not be read": an unread repository is asked again, a
  budget stop is waited out.
- **The secondary rate limit is not visible to `rate_limit`** — measured 2026-10-10,
  `gh` answered 403 on every call while the budget read 5000/5000.

⛔ **And two of them were blind to this fleet's own pages until v0.67.0**, which is
the same failure one level up: a tool that is right about its rules and cannot read
the subject answers confidently anyway.

- `tagcells` keeps the version a page states equal to each module's latest release.
  It looked for `[[params.modules]]` with `tag = "vX.Y.Z"`; the go-* landings are
  `[[params.repos]]` with `label =`. Against `go-fleettools.github.io/hugo.toml` it
  answered `0 cell(s)` and exited **2** — rightly refusing to call nothing a pass, and
  the reason thirteen labels dated in a comment above them had never been checked.
  When it first could look, **three of the thirteen were stale** and `fleettools`'
  own was twenty-seven releases behind.
- `pagetext` reported text **missing** on a page that carried it, six retries running,
  because of an apostrophe: Hugo prints `fleet’s` where the source said `fleet's`, and
  the match decoded entities but not typographic punctuation. Every English possessive
  and contraction was a false negative. A verification tool that says "not published
  yet" about something published is worse than none — the next person reads a real
  failure the same way.

## Flags: one dash became two {#flags}

Since **v0.53.0** all of the commands use [cobra](https://github.com/spf13/cobra), so
the canonical spelling is `--long` with POSIX short forms. The fleet had typed
`-dir`, `-apply`, `-org` for months, and pflag reads one dash as **one letter**, so
every one of those would have been refused: a single-dash compatibility shim in
`internal/cli` rewrites the old spelling and warns, which is why there was no flag day.
`cmd/flagcensus` is the measurement that drove it — 54 commands, 144 flags, 137 of
which needed the second dash.

⛔⛔ **And this repository's own workflows kept typing one dash for another year's
worth of runs.** v0.82.1 fixed **eight call sites** in two files — `vulngate -go`,
`vulnscan -selftest` twice, three more `vulnscan` lines, and `brandcheck -org` —
every one of them invoking a binary built from that very tree, so cobra, so the
shim was the only thing making them work. The shim prints a warning on every run
and announces its own removal: **a warning inside a green lane is a warning nobody
reads**, and the day the shim goes those lanes fail for a reason unrelated to
anything they measure.

Seven were found by another session and routed here through two lease notes. The
**eighth** was found by the test written for the other seven, on its first run, in
a file nobody had looked at — `TestNoWorkflowCallsOurOwnCommandsWithASingleDash`,
which enumerates `cmd/` rather than naming the commands, refuses to run if it sees
no workflow or implausibly few commands, scans only *after* the command's own token
(so `go build -o /tmp/vulnscan` is not a call), and skips comments — its first
version flagged a line of prose describing a hole its author had already removed,
and refusing what the protected thing accepts is a defect of the same rank as
missing one. `modfloor` v0.8.1 carries the narrow version of the same gate, because
its security lane had the defect too.

## The reusable workflows {#reusable}

Three checks here are **called by** the repository they guard, so they fail where
the change is made rather than in a report nobody opens — nine lines in the caller,
and ⛔ not `on: [push, pull_request]`, which both fire for a branch with a pull
request open and doubled every run on the first three organisations that copied the
snippet.

| | |
| --- | --- |
| `docs-current.yml` | does this organisation advertise what it has? Needs no secret: listing a public organisation is a public read |
| `brand-complete.yml` | does a brand repository carry what its README promises? Written because a brand repository has nothing to compile, so it had no checks at all and `ghmerge` rightly refused its pull requests |
| `go-floor.yml` (v0.82.0) | does this branch declare a `go` directive at or above the fleet's floor? |

⛔ `go-floor.yml` exists because the fleet-wide sweep **structurally cannot** catch
its case. `modfloor --baseline` answers *did anything get worse*, and a repository
*created* below the floor arrives as `new to the population`, where nothing says
whether it was ever clean. Measured over the five days to 2026-11-07: **0 regressed,
14 fixed, 61 arrived exposed** — the fleet's exposure went 262 → 314 without one
regression. Its `floor:` input is **required**, because a default would move under
every caller with no commit to review, which is the `go-version: stable` defect this
fleet spent a campaign removing; it pins the judging tool to a **tag** rather than a
branch, since a gate that fails builds must not change what hundreds of repositories
are held to with nothing reviewed; and `--offline` means it reads the working tree,
so there is no API call and no secret.

## `quietscan` {#quietscan}

`quietscan` is every organisation whose Renovate runner has **stopped**, **runs
and fails**, or **asks for a secret that does not exist**. `redscan` asks whether the
last run failed; a runner that stopped does not fail, it says nothing — so this asks
*when* it last ran, and then what became of it. **Sixteen runners were once reported
healthy while every nightly run failed in ten seconds**, which is the second half the
README added.

Since v0.66.0 it also reads the secrets a workflow *references* against the ones that
exist and reports `missing_secret` with the name, because the sixteen were all one
absent `RENOVATE_TOKEN` and an unset secret expands to the empty string — so the
action runs with nothing there and fails in ten seconds, which reads as *activity* to
any check that only asks whether runs happen. That check cannot live inside the
workflow: GitHub's documentation states that secrets cannot be referenced in `if:`
conditionals, and the documented workaround would put a credential in an environment
variable to learn something that needs no value. It found its first real one on
2026-11-07 — a runner created six days earlier whose secret was never placed, caught
in a day rather than the ten the sixteen took.

The README devotes sections to it: "What `quietscan` is, and the **seven** rules it
will not soften", attribution ("the property belongs to whatever COVERS the repository"),
"The trap `quietscan` exists to catch before it latches", "A failed call must never
become a verdict", "What one pass costs, and when not to run it", and "Two things
`quietscan` is not". Read them there before scheduling it beside another sweep.

## `openfixed`: open issues a merged pull request already addressed {#openfixed}

`openfixed --org A --org B` (or `--owner`) lists **open issues that a merged pull
request already addressed**, so they can be closed. For each owner it searches open
issues (the ones with a closing reference, `linked:pr`, first) and reads each issue's
timeline once. **CLOSABLE** means a merged pull request referenced the issue with a
closing keyword (or the timeline records it as *connected*) yet it is open; **REVIEW**
means a merged pull request mentioned it without a keyword, or the issue was reopened
after the merge, so a person decides. Issues whose pull requests are all unmerged are
not listed. The report states its denominator per owner (issues searched, timelines
read, issues skipped and why) and exits 2 if anything was skipped: a scan that could
not read is never reported as zero. It is a dry run by default; `--close` closes only
CLOSABLE issues, with a comment naming the pull request, and never a REVIEW one. It
stops before the REST budget falls under `--margin` (default 1000) so the fleet's
Renovate runners are not starved, and it paces the search API. Written against GitHub's
documented search qualifiers, the issue timeline events and the closing-keyword rules.
