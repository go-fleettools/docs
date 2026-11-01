---
title: "modfloor"
weight: 60
---

# modfloor {#modfloor}

The Go toolchain floor a module **declares**, against the floor a fix **requires**.
[Repository →](https://github.com/go-fleettools/modfloor) Latest tag: **v0.4.0**;
this page is written from its README at that tag. v0.3.0 added the `modfloor pseudo`
subcommand, [below](#pseudo); v0.4.0 splits the exposure by whether a consumer can
reach it, [below](#split).

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

```text
939 modules declare a floor below 1.27.3 …
   of those, 904 are PUBLISHED on the proxy (a consumer can install them)
   and 35 were never published (only a local build honours their floor).
   and 3 already carry the fix in their working tree but have not been RELEASED…
```

⛔ Worth saying plainly: the split was added because it **might** have shrunk the
campaign, and on this fleet it did not — 904 of 939 are installable. A measurement
earns its place when either answer would change what you do, not only when it gives
the answer you hoped for, so the README and this page record that it did not help,
beside the figures.

The counting is a named function rather than four variables inside the printer, so it
has a test and needs no network. The test plants all five verdicts — including
`toolchain`, which is **not** an exposure because it fails closed under
`GOTOOLCHAIN=local` — and asserts that published plus unpublished is the *whole*
exposure, so a row drifting out of both buckets cannot shrink the campaign silently.

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
