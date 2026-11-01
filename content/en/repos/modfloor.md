---
title: "modfloor"
weight: 60
---

# modfloor {#modfloor}

The Go toolchain floor a module **declares**, against the floor a fix **requires**.
[Repository →](https://github.com/go-fleettools/modfloor) Latest tag: **v0.5.0**;
this page is written from its README at that tag. v0.3.0 added the `modfloor pseudo`
subcommand, [below](#pseudo); v0.4.0 splits the exposure by whether a consumer can
reach it, [below](#split); v0.5.0 adds `--baseline`, [below](#baseline), and
corrects the population every earlier figure was counted over.

A repository can be green, released and built with a patched toolchain while still
shipping a vulnerable floor to everyone downstream. Three pins decide three
different things: `go-version:` in a workflow builds the artefacts; the toolchain
recorded in a binary says what one installed executable was built with; and the `go`
directive in the **published** `go.mod` is what a consumer honours under
`GOTOOLCHAIN=local`. Only the third is the exposure, and no green check looks at it.

## Install and usage {#install}

```sh
GOBIN=~/.local/bin go install github.com/go-fleettools/modfloor@latest
```

```sh
modfloor --dir ~/Documents/VCS/GIT/localhost --owners 'go-*' --floor 1.27.3
modfloor --bins ~/.local/bin --floor 1.27.3
modfloor --floor 1.27.3 github.com/go-fleettools/fleettools
```

Verdicts, kept apart because they call for different work:

| Verdict | Meaning |
| --- | --- |
| `ok` | the `go` directive is at or above the floor |
| `toolchain` | the directive is below it, but a `toolchain` line pins higher: the build fails closed under `GOTOOLCHAIN=local`. Not an exposure |
| `BELOW` | the directive is below the floor and nothing pins higher: **the exposure** |
| `no-go` | no `go` directive at all |
| `unreadable` | a directive that is not a Go version |

Exit status: 0 clean, 1 at least one module exposed, 2 the measurement could not be
made.

## What it reads, and what it refuses {#reads}

It reads the **published** `go.mod` from `proxy.golang.org`, where a consumer reads
it too, not the working tree (which may hold an unreleased fix) and not a workflow.
It spends no GitHub API budget and needs no credential. `--owners` is a declared
population, and every number in the report is about that set; a module in no git work
tree, or under another owner, is set aside and counted. A measurement that found
nothing **refuses** rather than reporting a clean fleet.

⛔ And **being in our work tree does not make the module ours** — the half that was
missing until v0.5.0. `remote.origin.url` answers "whose repository is this", not
"whose module is this", and a repository that carries other projects' sources put
540 of them into this fleet's figures. A module whose path is not the repository's
own path — `github.com/<owner>/<name>`, or a subdirectory or `/v2` of it — is set
aside as `published by another project`.

Two traps it encodes: `go/version` accepts `go27.2` (a typo that compares above every
real version), so the floor is validated; and `modfile.ParseLax` does not fill
`File.Toolchain`, so the directive is recovered from the syntax tree.

## The exposure, split by who can reach it {#split}

One number over both populations cannot be prioritised. A module on the proxy
imposes its floor on everyone who runs `go install`; one that was never published
imposes it only on somebody building from a clone. Those are a campaign and some
housekeeping, and until v0.4.0 the report added them together. A third line counts
the modules whose working tree is already at the floor and whose published module is
not — the `fixed locally, UNRELEASED` state, where the pull request is merged and the
work is **not finished**, because the consumer reads the published `go.mod` and not
the branch. This tool exists because those two differ, so that state is counted by
name instead of waiting in a `NOTE` column somebody would have to read 939 rows to
find.

Measured 2026-11-01, with the population printed beside it, because a count is only
as good as the set it counts over:

```text
population: owners go-*,tannevaled — 1779 go.mod seen, 348 kept; set aside:
410 out of scope, 43 in no work tree, 540 published by another project,
0 without a module directive, 0 unreadable
348 modules against floor 1.27.3:  ok=86  BELOW=260  no-go=2
⛔ 262 modules declare a floor below 1.27.3 …
   of those, 262 are PUBLISHED on the proxy (a consumer can install them)
   and 0 were never published.
   and 6 already carry the fix in their working tree but have not been RELEASED…
```

⛔ Worth saying plainly: the split was added because it **might** have shrunk the
campaign, and it did not — all 262 are installable. A measurement earns its place
when either answer would change what you do, not only when it gives the answer you
hoped for.

⛔⛔ **And the figure this page carried for a day was wrong.** It said *904 of 939*.
That count was true over a population that included **540 modules other projects
publish**, sitting inside our own work trees: `sigs.k8s.io/yaml`, `cel.dev/expr` and
twenty more, all inside `tannevaled/pantry`. Every `remote.origin.url` said
tannevaled, so every one passed the owner test, and nobody here can raise the floor
of `sigs.k8s.io/yaml`. v0.5.0 sets them aside and counts them on the population
line. The real campaign is **262** modules.

The counting is a named function rather than four variables inside the printer, so it
has a test and needs no network. The test plants all five verdicts — including
`toolchain`, which is **not** an exposure because it fails closed under
`GOTOOLCHAIN=local` — and asserts that published plus unpublished is the *whole*
exposure, so a row drifting out of both buckets cannot shrink the campaign silently.

## `--baseline`: a gate for a campaign that cannot be finished in one day {#baseline}

⛔ **A report that is red for ever is not a gate.** 262 published modules declare a
floor below 1.27.3, so `modfloor --floor 1.27.3` exits 1 every time it runs and will
keep doing so for as long as raising them takes. A lane in that state is read as
broken, then ignored — and the day a *new* module lands below the floor, nothing
changes colour.

```sh
modfloor --dir … --owners 'go-*' --floor 1.27.3 --write-baseline floor.txt
modfloor --dir … --owners 'go-*' --floor 1.27.3 --baseline floor.txt
```

`--baseline` exits 1 only for a module the recorded set does not have. The file is
plain text so a tightened baseline has a readable diff, and it names the floor and
the mode it was taken against.

| Refused | Why |
| --- | --- |
| no `floor` line, or another floor | every verdict changes the day the floor moves, and it moved three times in four days (1.27.1 → 1.27.2 → 1.27.3). A set of module names says nothing without the floor it was judged against |
| another `mode` | `--offline` records what the working trees declare, the default what the proxy serves: different answers about the same modules |
| `--baseline` **and** `--write-baseline` | recording the answer and then judging it against itself always passes, which is a gate that cannot fail |

And the distinction a subtraction gets wrong: a module that **left the population**
— clone deleted, renamed, moved out of the owner globs, or simply not walked —
leaves the exposed set exactly as a repaired module does. It is reported as
`NOT MEASURED THIS TIME (not a fix)` and counted apart, because adding the two
together is how a campaign declares victory by losing track of its subjects.

⭐ The baseline is also what caught the wrong population above. The report prints
348 rows and a summary, and nobody reads 348 rows; the baseline writes the exposed
set to a file as a sorted list of module paths, where `cel.dev/expr` two lines from
the top is impossible to miss.

⚠ Where it should run is **not settled**: a baseline is a statement about one
machine's clone population, so committing one in the repository would make its lane
depend on that machine.

## `modfloor pseudo`: dependencies that were never released {#pseudo}

```sh
modfloor pseudo [--dir DIR] [--input FILE] [--fleet-prefix PREFIX ...]
```

```sh
modfloor pseudo --dir ~/Documents/VCS/GIT/github.com/go-llmflow/gateway \
  --fleet-prefix github.com/go-volumes/ --fleet-prefix github.com/go-authn/
```

```text
MODULE                                     VERSION                             FLEET
github.com/go-volumes/interface            v0.0.0-20261026102434-e5b421ad6fae  FLEET
google.golang.org/genproto/googleapis/rpc  v0.0.0-20260928230214-8a89bd6388cc
2 of 120 modules at a pseudo-version, 1 of them the fleet's
```

It lists every module of a build graph that is required at a **pseudo-version**, that
is at a commit instead of a tag. It reads `go list -m -json all`, run with
`GOWORK=off` in `--dir`, or a saved copy given with `--input`. A replaced module is
judged by what is actually built. Modules matching a `--fleet-prefix` are marked
`FLEET`; without any prefix, every module counts as the fleet's.

**Why it matters.** A pseudo-version names a commit that semver says nothing about,
and it stays in the module proxy for ever. On 2026-10-26, go-llmflow/gateway depended
this way on two fleet modules that had never been tagged.

| Exit | Meaning |
| --- | --- |
| 0 | no fleet module is at a pseudo-version; third-party modules at one are listed but do not fail the check, because their upstream decides whether to tag |
| 1 | at least one fleet module is at a pseudo-version |
| 2 | the graph could not be read; a graph with no dependency also exits 2, since it was not measured |

The README names its siblings: `vulnscan` and `vulngate` for the advisories
themselves, `gopin` for the CI pin, and `staletools` for installed binaries.
