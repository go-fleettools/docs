---
title: "fleettools"
weight: 10
---

# fleettools {#fleettools}

Sweeps across about 1800 repositories and about 320 organisations: which default
branches are red, which runners stopped or run and fail, which modules are ahead of
their tag, which foreign judges the CI never installs, and which tests exist without
ever running. [Repository →](https://github.com/go-fleettools/fleettools) Latest tag:
**v0.161.0** (`740afd7`, read on 2026-11-22 with
`git ls-remote --tags … refs/tags/v0.161.0`, which is also how `tagcells` resolves it —
the commit is read from the tag, never carried over from the previous one). The prose below was
written from the repository at **v0.67.0**, plus the two v0.82.x sections and the
`modtidy` section at the end written from the change itself — the command inventory under `cmd/`,
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

**102 directories under `cmd/`**, each with a `main.go` (counted 2026-11-20 with
`git ls-tree -d cmd/` on `a57b2c0`, which is the released head of v0.141.0; it
was 101 on 2026-11-19 at `85ef23c`, the merge commit of
[#367](https://github.com/go-fleettools/fleettools/pull/367), 94 on
2026-11-16 at `62dadb3` for v0.128.0, 88 on 2026-11-15 at `a67ac44`, 86 earlier that day on
`4235455`, 79 on 2026-11-10 at `69b51b87` for v0.104.0, 72 on 2026-11-09 at
`5d0fae1` for v0.94.0, 65 on 2026-11-07 at `06dfd81`, and 64 at v0.81.2):

```text
abbench acdefault actionpin aptneed archrun binnames blobpurge branchscan brandcheck
brief buildscan buildtag bzcomment clonescan copycheck covblocks cronscan ctxgate
dashfix declines diffrefs docmd docscan docscheck docsversions docsym editurl envcheck
failset flagcensus flakescan fleetverbs ghadvisory ghissue ghpr ghrel ghrepo ghrun
glmerge gofloor gopin gorequires gotest gvlscan handlayout hostload i18ncheck judgescan
landingkitdist landingmigrate landingorgs landingrollout landingrollsum langcheck
linkcheck linkjobscan mantracheck mdpdf mkdocs2hextra modelgate modrename modtidy needread
openfixed orgfile osvm pagesbuild pagesbuildadd pagesbuildcampaign pagescheck pagespublish
pagesretire pagetext paritycheck pinscan pngeq priorart privscan prmerge prrefresh
prscan pullresidue qemufix quietscan ratewait recur redscan relverify renovatedup
replystamp runson secondorigin secretscan sitelinks sitemigrate staleref symfind tagcells
tagscan testscan testtime testwhy tinydur treecmp truncspill unarc viaonly vmkey vscan
vulngate vulnscan waitload wasmsize wfaudit
```

⚠ The list above was re-read on 2026-11-22 from
`git ls-tree -d --name-only v0.161.0 cmd/`, name by name. The **count** in the
first sentence was NOT re-taken with it: no program compared the two this time, so
the 102 stays dated 2026-11-20 rather than being replaced by a number counted by eye.
Eleven names were missing from the previous list and are added here: `actionpin`,
`archrun`, `ctxgate`, `dashfix`, `declines`, `envcheck`, `ghadvisory`, `handlayout`,
`orgfile`, `osvm` and `renovatedup`. Apart from `renovatedup`, which is described
below, they are named and not described.

The one added since the 101 is `modelgate`
([#368](https://github.com/go-fleettools/fleettools/pull/368), 2026-11-20, by
another session; *"refuse a sub-agent launched with the wrong model"*), named by
`git log --diff-filter=A 85ef23c..v0.141.0 -- 'cmd/*/main.go'` — 101 + 1 = 102 —
and, like the six below it, named and not described: its code was not read for
this page.

The seven added since the 94 are named the same way, by
`git log --diff-filter=A 62dadb3..85ef23c -- 'cmd/*/main.go'`, and the
arithmetic reconciles — 94 + 7 = 101 — against the list above, which was
checked against `git ls-tree` by a program rather than by eye. ⛔ **One** of the
seven this page describes below, `flakescan`, and **six it does not**: `vmkey`
([#339](https://github.com/go-fleettools/fleettools/pull/339)), `testtime`
([#345](https://github.com/go-fleettools/fleettools/pull/345)), `acdefault`
([#347](https://github.com/go-fleettools/fleettools/pull/347)), `diffrefs`
([#349](https://github.com/go-fleettools/fleettools/pull/349)), `binnames`
([#356](https://github.com/go-fleettools/fleettools/pull/356)) and `waitload`
([#357](https://github.com/go-fleettools/fleettools/pull/357)), each merged by
another session between 2026-11-17 and 2026-11-18. Their titles are in the
pull requests; their code was not read for this page, and the rule above is the
reason they are named and not explained.

The six added since the 88 are named by `git log --diff-filter=A` over
`cmd/*/main.go` rather than by comparing two lists by eye, and the arithmetic
reconciles — 88 + 6 = 94. ⛔ Two of them this page describes below, and **four
it does not**: `landingkitdist`, `landingorgs`, `landingrollout` and
`landingrollsum` arrived together on 2026-11-15 in
[#326](https://github.com/go-fleettools/fleettools/pull/326) as the landing-kit
rollout, and nobody has read their code for this page yet. Naming them in a
counted list is a measurement; describing what they do without reading them
would be the defect this fleet keeps writing down — a true observation with an
invented mechanism — so the gap is stated instead.

`viaonly` (v0.126.0, 2026-11-15) answers two questions about a package: which
**calls** bypass the wrapper they were supposed to go through (`--wraps/--via`),
and which **exit points** never ask the question the function exists to ask
(`--asks/--in`). The second rule needed a second shape rather than a flag,
because an early `return` does not read as an exit and a `return` is not a call.
It credits only what is **certainly evaluated**: of `A && B` only `A` is, a case
list is evaluated in order so only its first expression is, a clause reached by
`fallthrough` evaluated none of its own, and a `defer`, a `go` statement or a
function literal is never credited. ⛔ Its own first version let through exactly
the defect it was written for: it credited *"this statement contains a
question"*, so a question asked inside one `switch` case marked every later exit
as having asked it — the defect's second round, with the suite green and
coverage full. What caught it was recalculating a twenty-row expectation table
by hand, where **five of the twenty were wrong**; three further review rounds
found ten real defects in it, each time against a green suite.

`ratewait` (v0.128.0, 2026-11-16) blocks until a named GitHub budget is at or
above a threshold, and exists because nothing in the fleet could **wait**. The
hourly budget is per *user* with about twenty-five sessions drawing on it, so a
program meets a 403 it did nothing to cause, and `fleet.Throttled` answers
`false` for *API rate limit exceeded* on purpose — that budget will not refill
inside any backoff — which correctly ends the retry and leaves nobody able to
come back at the reset. It sleeps to the reset the API reports rather than on an
interval, because a poll loop spends the budget it is waiting for. ⛔ **Four
rounds of review each found its loop unbounded in a new way, and the result is a
rule about waiting rather than about one command: a predicate over a value the
remote reports is not a bound you control.** *Strictly later than the last
reset*, *later by at least the margin* and *strictly in the future* were each
defeated by a reset that moves with the clock — the last measured at five
readings in twelve seconds, about 1300 under a 65-minute deadline. What bounds
it is a **count** of readings derived from the window the API's own body shows
(the farthest reset among fifteen resources is 59m59s), three plus one per hour
the deadline allows; flooring that division refused a legitimate wait at
`--timeout 59m`, and its own test had pinned the undercount **as the
specification**, so a mutation raising the count was reported *caught* by an
expectation that had moved with the defect. A `+ unit - 1` ceil also overflows
`int64` at the largest duration the flag parser accepts, which turned the
longest wait askable into an instant refusal.

`dashfix` (v0.148.1, 2026-11-20) fixes one line of an organisation's Renovate
runner config on the branch of a red Renovate pull request, then waits for CI
and merges it: `dependencyDashboard: true` becomes `dependencyDashboard: {
enabled: true }`. Upstream Renovate migrated that key to an object, and the
validator these repositories run (`renovate@latest --strict`) now exits 1 on the
boolean, so every Renovate pull request in them was red for a reason its own
diff does not show ([#305](https://github.com/go-fleettools/fleettools/issues/305)).
Targets are spelled `owner/repo#number` and handled one at a time. For each, the
REST budget must hold `--reserve` requests (default 1000) and the trailing hour
fewer than `--max` merged pull requests (default 20), or the run **stops**; the
pull request must be open with a *failed* check (green or pending ones are
skipped, never touched); `agentsync claim` must succeed (a repository somebody
else holds is skipped and listed); a repository whose workflows carry
`RENOVATE_VERSION` is skipped and listed, because aligning that pin is a
different change. It then commits, pushes with `gitpush`, waits with `prwait`
and merges with `ghmerge` only when `prwait` says green. Any command that fails
or refuses stops the whole run with status 1 and is never retried another way;
a skip is not a failure. `--dry-run` judges and edits nothing. Exit status: 0
every target merged or skipped, 1 something was refused, 2 the run could not
start or stopped at a limit.

`flakescan` (merged 2026-11-19 in
[#367](https://github.com/go-fleettools/fleettools/pull/367), released in
v0.141.0) counts how often a
**named test** has failed across a repository's CI history, by lane, with a
denominator. It exists because go-crdt/collab#204, one flake, had its sweep done
by hand three times over three populations — commits on `main`, *red* runs on
`main`, the pull request somebody happened to be on — and got three answers.
⛔ **It reads runs × attempts, not runs.** Run `39069166645` of that repository
is `attempt 2, conclusion success` on a pull request, and its first attempt
holds a failure of the test: a flake is precisely the failure that gets re-run,
and a re-run repaints the run green while the evidence stays in attempt *n−1*,
so a census over runs whose conclusion is `failure` cannot see what it counts
and "N in M runs" is a **floor**. It also reads a failed *step* inside a green
job (`continue-on-error` leaves the job green), every page of a job list (a
short read is an error rather than a smaller answer), and only the logs of jobs
that failed. Zero occurrences **with anything unread exits 2**, never 0, because
GitHub discards logs on a retention window. First run, over the whole window of
that issue — 257 runs, 265 attempts, 6296 jobs, 66 logs: **3 occurrences across
two lanes** (`arch amd64` 2, `test (ubuntu-latest)` 1), the three the issue
already recorded and no fourth, with one listing unread and named. ⛔ Its
mutation sweep found the suite reaching the **real GitHub** for 4 min 25 s with
a refusal deleted, because the flag table's only protection from the network was
the guard under test; the reader is a variable the tests replace now.

`secretscan` (v0.157.0) finds the daemons that take a secret as a **plaintext
value**: a config-struct field, a cobra or pflag flag, an `os.Getenv` name, a
systemd `Environment=` or `ExecStart` flag, a launchd plist, instead of by
reference (a go-authn/secretsource `*_from` such as `systemd-creds:name`). A
name is judged by its words, never a substring: `token_file` is a path,
`password_from` a reference, `max_tokens` and `key_file` are not secrets, and a
flag whose default is a path (`~/.github-token`) or whose usage says file or
path is a path. `secretscan sweep --root <dir> --json out.json --md out.md`
reads every clone, opens its report with the denominator (clones seen, read,
skipped; files read, unread), counts daemons and findings per repository by
canonical origin URL so the worktrees of one repository count once, and exits 2
when anything was skipped or unreadable: an unread clone is NOT READ, never
clean. `secretscan repo [--waive "<reason>"] [--all] CLONE` is the gate for one
repository (exit 1 on a plaintext input of a daemon). A flow into `os.Open` or a
`*From` reader is not followed, and a `main` that only calls a library server is
found through its unit file or not at all.

`renovatedup` (v0.159.0; its GraphQL budget guard in v0.161.0) finds the
repositories that apply their organisation's Renovate preset **twice**. When
`<org>/.github/config.js` sets `inheritConfig`, Renovate merges the inherited file
into every repository and then also reads the repository's own config, so a
`renovate.json` whose `extends` names that same file registers every custom manager
twice (measured on go-crdt, go-crdt/.github#45: the dashboards went from `regex (4)`
to `(2)` once the stubs were removed). `renovatedup [--orgs ORG,...]` reads each
organisation in **one GraphQL query per 100 repositories**, with the `.github`
`config.js` and every candidate config file in Renovate's search order, and reports
`STUB` (the file holds only that `extends`: remove the file), `MIXED` (other rules
too: remove only the entry) or `UNREAD` (a JSON5, truncated or unparsable file, or
a failed organisation query). Exit 0 is clean, 1 a duplicate, 2 something unread,
and 2 outranks 1. Measured with v0.161.0 on 2026-11-22: go-crdt 1 STUB
(`go-crdt/brand`, a recorded exception), go-composites 34 STUB out of 34
repositories. Since v0.161.0 the shared budget guard charges a GraphQL call to the
GraphQL budget, so the sweep still runs when the REST budget is spent.

The two added since the 86, both on 2026-11-15, are `editurl` and `recur`.
`editurl` asks whether a Hextra site's *"Edit this page on GitHub"* link reaches
the page's own source, and reads the **repository** rather than the URL: GitHub's
`/edit/` route authenticates before it resolves a path, so an anonymous request
to an existing repository answers `302` to a login page whatever the path is, and
a fetcher cannot tell a correct href from a broken one. Measured over 1318
checkouts: **75 of the 97 sites that render the link do not resolve** — 64 name a
repository that does not exist (`github.com/<org>/edit/…`), 7 name the
organisation's *code* repository rather than its docs one, and 4 double the
language (`…/content/en/en/page.md`). `recur` refuses to publish while a lesson
that recorded a recurrence has no gate.

The seven added since the 79 are `aptneed`, `failset`, `modtidy`, `paritycheck`,
`replystamp`, `testwhy` and `tinydur`. Four of them are gates against a *rule
that was written and then not followed*: `replystamp` for a reply with no
timestamp, `paritycheck` for a body that parks a performance change on bit
identity, `modtidy` for the nested `go.mod` nobody tidies ([below](#modtidy)),
and `failset` for a failure list compared by count instead of by label.

The seven added since the 72 are `copycheck`, `docmd`, `gotest`, `pullresidue`,
`runson`, `secondorigin` and `staleref`. Three of them exist because a *check*
could not be trusted to run: `gotest` refuses to let `go test -run` report `ok`
for a pattern that matched nothing, `staleref` finds the names a rename left
behind, and `copycheck` the text that was duplicated instead of shared.
`runson` was the last fleet tool living outside a repository — a Go program with
tests in a plain directory with no remote — which is the one thing a tool here
may not be.

⛔ **This count has now gone stale four times**, which is worth keeping on the
page rather than quietly correcting. It said **55** for four days, then **59**
against a repository that held 64, then **79** against a repository that held
86 — five days, seven commands, and again nothing recounted it — and then **86**
against a repository that held 88, which went stale **the same afternoon it was
written**: two commands merged between the recount and this edit. That is the
shortest life any of these figures has had, and it is the one that makes the
point best, because the author of the figure was still at the keyboard. Each time the
figure was right when it was written, which is exactly what makes it hard to
doubt: a number on a page carries no sign of when it was true. The remedy that works is the
one used here: cite the **commit** the count was taken on, so the claim carries
its own expiry — and nothing checks it automatically. ⚠ This paragraph used to
say `docsversions` was "the tool for the version a page claims"; it is not, and
reading its `--help` says so — it builds every published version of a versioned
site and publishes them, which is what it is described as further down this page.
No command in this repository judges whether a figure on a page still holds, so
the commit citation is the whole mechanism, and
`docscan -fail-on-drift` reported **0 drift** throughout every episode — correctly
by its own definition, and blind to a version asserted in prose. The gate for
that is filed as item 1 of
[fleettools#201](https://github.com/go-fleettools/fleettools/issues/201) and is
still unbuilt; the fourth occurrence above is its fourth witness. What makes it
buildable rather than a wish: `tagscan` already resolves each repository's real
latest tag, so the comparison exists — what is missing is reading the claim out
of the page, and a negative control, because a page that deliberately documents
an older tag is legitimate and must not fire.

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
  A surface it could not read holds the gate shut (status 2) **even when another
  surface found something** — and that clause was the defect rather than the design:
  `found` was tested before `unread`, so a branch match returned "already covered" on
  a run whose tracker had answered HTTP 422 and been read by nobody, printing the
  warning above a verdict that said the opposite. The shut gate includes a GraphQL
  refusal, which arrives as HTTP 200 with an `errors` array: read past that and a
  refused query becomes an empty branch list, which is a clean gate on a surface never
  read.

  The search is **split at six terms**, because GitHub allows five `AND`/`OR`/`NOT`
  operators and seven OR-ed clauses answered HTTP 422 outright — so the subjects that
  silently lost the tracker half were the carefully written ones, which are the
  subjects most likely to have prior art. Terms are batched six to a query and the
  results unioned; one failed batch is the **whole surface unread**, not a partial
  answer. Splitting costs the single global ranking, since GitHub ranks within a
  query, so batches are emitted in term order and each keeps the order the search gave
  it — nothing re-ranks them, because a scoring pass of its own was measured against
  GitHub's relevance and came out worse.
- **Does the CI actually test it?** `testscan`, `judgescan` (tests that look for an
  external tool the CI never installs — a foreign judge that exists in the source and
  has never once run), `covblocks` (the blocks a coverage profile never entered, because
  `go tool cover -func` rounds to one decimal and a 100 % gate passes at 99.95 %),
  `buildscan`, `buildtag`, `flagcensus`, `runson` (which **runner images** the
  workflows ask for, counted per repository and not per file: 1226 of 1247
  checkouts with workflows stand on `ubuntu-latest`, which upstream is moving to
  Ubuntu 26.04 between 2026-10-19 and 2026-11-19 — inside that window a red CI
  can be the *image* moving while the repository did not). ⛔ It reads checkouts
  and makes no API call, and the price of reading a disk is stated rather than
  disclaimed: it **measures** how many of the checkouts it read are behind their
  remote — 842 of 1247, furthest 188 commits, with 405 more never fetched within
  the horizon, so not one is confirmed current — and says which way that makes it
  wrong, because a checkout predating a newly added workflow makes the count a
  floor and an absence read off a stale disk looks exactly like a real one.
- **Is it safe?** `vulnscan`, `vulngate` (the per-repository half), `gofloor` (which Go
  toolchain a repository can be built with, and whether CI would notice a vulnerable
  one), `wfaudit`, `blobpurge`, `needread` (the `DT_NEEDED` entries of every ELF under a
  tree, refusing to report unless a symbol it was told to expect is actually there),
  `symfind` (whether an ELF *exports* a dynamic symbol, rather than merely consuming it).
- **Is it advertised, and is the page what it builds?** `docscan`, `docscheck` (which
  generator a published URL was built with), `pagescheck` (whether a Pages site serves
  what its repository builds), `pagetext` (whether a published page now carries the text
  we wrote), `sitelinks` (whether every local link of a built site leads to a file of
  that build), `editurl` (whether a Hextra site's *"Edit this page on GitHub"* link
  reaches the page's own source — it reads the repository, because that link's status
  code cannot tell a correct href from a broken one), `linkcheck` (whether every **external** link a documentation source names
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
means a merged pull request mentioned it with *no* keyword at all, or the issue was
reopened after the merge, so a person decides; **REFS** means a merged pull request
wrote `Refs` (or `Ref`), which says the issue stays open **on purpose**. Issues whose
pull requests are all unmerged are not listed. The report states its denominator per
owner (issues searched, timelines read, issues skipped and why) and exits 2 if anything
was skipped: a scan that could not read is never reported as zero. It is a dry run by
default; `--close` closes only CLOSABLE issues, with a comment naming the pull request,
and never a REVIEW or REFS one. It stops before the REST budget falls under `--margin`
(default 1000) so the fleet's Renovate runners are not starved, and it paces the search
API. Written against GitHub's documented search qualifiers, the issue timeline events
and the closing-keyword rules.

### Why REFS is a third word and not a flag {#openfixed-refs}

Until 2026-11-22, REVIEW meant two unrelated things: *nobody has said what becomes of
this issue* and *somebody said, in writing, that it stays open*. The fleet-wide merge rule made the second one **mandatory** — `ghmerge` refuses
a merge whose body mentions an issue and classifies it as neither closed nor kept — so
every correctly written `Refs #N` became permanent REVIEW output. One organisation's
REVIEW held **37 of its 64 open issues**, in a bucket whose name means *a person
decides*, and nobody reads such a bucket (go-fleettools/fleettools#204).

A REFS row is still **printed**, because an author can write `Refs` and be wrong, and
an issue reopened after the merge still deserves a look. What changed is that it is
counted apart and sorted **last**, after the rows that still need a decision — the
order carries the meaning, and it is the only part of this that no count could hold: a
hand-applied mutation swapping the two ranks left every total correct and moved only
the order, and exactly one assertion went red.

⛔ **`See #N` is not one.** The gate's keywords are close/fix/resolve and `ref(s)`, and
`see #12` is the phrasing the merge gate exists to refuse — it reads as a decision and
is not one. The issue asked for `Refs` and `See` to be read alike; doing that would
have silenced the audit for exactly the sentence the fleet rejects at merge time. Both
spellings are negative controls in the test table now.

⛔ **The reading is the merge gate's own parser**, `issuegate.Parse`, which answers
*closes*, *refs* or *bare* in one call. A private second regexp for `Refs` would be a
new way for the audit and the gate to disagree — which is how a heading
“## Also filed, not fixed” once bound to a reference two lines below it.

## `privscan`: a tool and a door against publishing a private file system {#privscan}

On 2026-11-09 three issues were filed whose bodies each carried
`/Users/<account>/Documents/<project>/<file>.tex` — a home directory, an account
name, a directory layout and the name of an unpublished document. Nobody noticed
until the owner of that machine asked whether their name had been leaked. The
rule set afterwards names **three** things, not one: no leak of a name, nor of a
path or a file name, on a user's file system.

⛔ **An edit does not undo it.** GitHub keeps the original under *edited* for
anyone who opens the history, so the only complete remedy is to delete and
re-create — which is why `ghissue --delete` exists, and why the gate has to run
*before* the send rather than after.

`privscan body.md` checks a file; `--fix` rewrites it; `--repo owner/name` and
`--org NAME` read what is already published, **comments and titles included**,
because a code search does not index comments reliably — the first sweep of this
leak found one occurrence and could not have found a second in a comment. Three
paginated repository-wide calls read everything, against one call per issue. A
repository whose read failed exits 2 and is never printed as clean.

A finding prints its line, column and kind but **not the text it matched**
unless `--show` asks: a refusal is printed where the body was going, and that is
sometimes a public CI log.

### The gate, inside `ghissue` and `ghpr` {#privscan-gate}

Since v0.129.0 the gate lives in `internal/privgate`, shared by `ghissue` and
`ghpr`, and `ghpr <owner/repo> create --head BRANCH --title T --body-file F`
(`--base-branch` is the target; `--base` stays the API URL) opens a pull request
through it: it also refuses a head with no commits ahead, a head that already has
an open pull request, and a body whose issue references lack `Closes`/`Refs`.

Every body and title `ghissue` and `ghpr` send passes `privpath` first, in `readBody` and
`checkTitle` — the two functions every path through the command goes through —
with the waiver as a **parameter** of both, so no future subcommand can read
outgoing text without the compiler asking what to do about a disclosure. A gate
reached by remembering to call it is not a gate.

`--allow-private-path "<why>"` takes a **reason and has no default**: a boolean
waiver is one keystroke and gets passed by reflex, while a reason has to be
written, and writing it is the moment to notice there is nothing to write. It
**stamps** — the reason is appended to the published text, so a waiver is a
statement made in public rather than a switch flicked in private. **No waiver
covers a title**: a title cannot carry the stamp and is republished in every
listing, notification and cross-reference that an edit never reaches.

### What the first sweep actually measured {#privscan-measured}

509 findings over 60 repositories — and almost all of them were the instrument,
not the fleet. Reading **one** real body settled it, where no reading of the
count could have:

| run | findings | what reading the text removed |
| --- | --- | --- |
| 1 | 509 in 67 texts | a package manager's `/home/.zpkg/…` prefix read as a person called `.zpkg`, every package name along it then hunted as that person's document; and a TeX `\\` — a line break in LaTeX, an escaped backslash in every Go string — matched as a Windows share whose one-letter name was hunted through all 60 repositories |
| 2 | 126 in 51 texts | `/Users/Shared`, which is everyone's; and `Home/End/Space`, a list of **keyboard keys**, read as the home directory of a person called `End` |
| 3 | 45 in 6 texts | — |

The single real disclosure was the same in all three runs: the one the first
sweep had already found. A three-figure count from a new instrument on a large
corpus measures the instrument.

⛔ Hence `--accounts`, which answers the question a count cannot: **whose** file
system was named, and whether this machine's is among them. The verdict is a
sentence, not a number, and a clean sweep prints it too — silence after a sweep
reads as *clean* and cannot be told from a command that printed nothing.

⛔ And hence four sentences for four states of "who am I", which CI had to
teach: `--no-self` (not checked), an unreadable home, a **generic** account
(`/home/runner` identifies nobody, so there is nothing of its own to disclose),
and a real one. The first version printed "could not be read" for the generic
case, which was false — an instrument whose two outcomes read alike measures
neither — and computed a verdict about an account `--no-self` had excluded.

Two design costs are stated on purpose. A generic account's path is skipped
whole, tail included, so a document name under `/home/user/` goes unreported;
and nothing below a dot-directory is hunted bare, because
`/Users/<account>/.pkgx/llvm.org/v22.1.8/bin/clang-22` taught the scanner
`llvm.org` and `v22.1.8` — public package coordinates named six times in the
prose that *was* the report, which a redaction would have eaten.

## `modtidy`: every `go.mod` of a clone, and the two answers it refuses to guess {#modtidy}

Written from the change at **v0.123.0** (`4235455`), not from the README.

`go mod tidy` at the root **does not enter a directory that has its own
`go.mod`**. Three recorded occurrences, each a root that was tidied and tested
green before the CI went red with *"updates to go.mod needed"*: `go-widgets/toolkit`
twice, through its `rougelex/` sub-module, and `go-numerics/integrate` through
`bench/`. So `modtidy [dir]` finds every `go.mod` beneath a clone and judges each
one with `GOTOOLCHAIN=local GOWORK=off go mod tidy -diff` — the CI's own
constraint, because `GOTOOLCHAIN=auto` fetches whatever a directive asks for and
hides the problem, and a `go.work` file would let a sibling satisfy a requirement
the CI never sees. `modtidy --hook` is the PreToolUse form: it acts only on a
command position `gitpush` or `git push`, and denies the push naming the module
and the fix.

A module is reported `tidy`, `untidy` with its diff, `UNREAD`, or `skipped`.

⛔ **UNREAD is never clean**, and the two classes below exist because the command
twice gave an answer where it could not know. Both were measured on 2026-11-15.

### A module with no Go source cannot be judged by a tidy {#modtidy-nosource}

`go-fft/docs` is a Hugo site whose only requirement is its theme. Running the
remedy the hook itself printed — *"`go mod tidy`, commit go.mod and go.sum"* —
empties both files and the site loses its theme. A gate that instructs the next
session to break the repository is worse than no gate.

In a module with **no Go source**, a tidy can only ever *remove* requirements, so
its verdict carries no information. Such a module is `skipped`, with the reason
named, decided before `go` runs at all — no toolchain, no network.

Three details are the fix rather than decoration:

- **A Hugo configuration is reported, never what decides.** `go-fft/docs` has no
  `[module]` block; it mounts its theme with `theme = ["…"]`. A rule keyed on that
  block would have missed the one repository it was written for.
- **A nested module's sources never answer for its parent**, or a Hugo site with a
  small Go tool beside it reads as Go code.
- **A file behind a build constraint still counts.** A `//go:build tools` file
  holding a blank import is the documented way to keep a requirement no package
  uses, and skipping such a module would have hidden two dead `go.sum` lines this
  gate correctly found in a landing.

⚠ And the pattern the fleet already documents **cannot** repair such a repository
instead: `go mod tidy` reports *"module … found, but does not contain package
github.com/McShelby/hugo-theme-relearn"*. A landing kit is an importable Go
package; a Hugo theme is templates and CSS that merely carry a `go.mod`. There is
no package to import, so no Go code can hold that requirement — and nothing in the
shape of the two repositories announces the difference.

### A directive floor above the installed toolchain is not a verdict {#modtidy-floor}

`GOTOOLCHAIN=local` is the right constraint to reproduce, but on a host whose
installed Go is older than a module's `go` line it cannot build the module at all —
and in the hook, UNREAD means *allow*. The gate had therefore gone silently off for
every repository raised to `go 1.27.4`, **this one included**:

```text
UNREAD  .: go: go.mod requires go >= 1.27.4 (running go 1.27.3; GOTOOLCHAIN=local)
```

The check now retries once with the toolchain the directive **names**, which
reproduces the CI rather than loosening anything: the runner there *has* the version
the directive requires, which is the only reason `GOTOOLCHAIN=local` succeeds for it.
Deliberately not `GOTOOLCHAIN=auto`, which leaves the version used unstated — an
instrument whose version is implicit is not an instrument. `GOWORK=off` is untouched.
The report names the toolchain that answered, so a verdict reached under the
directive's version can be told from one reached under whatever the host happens to
have installed.

⛔ A broken `go.mod` is still UNREAD, never `skipped`: a skip is a clean answer, and
"holds no `.go` file" has not read the file that is wrong. The repository's own
corpus caught that during the change — a `go.mod` holding garbage has no Go source
beside it either.
