---
title: "modfloor"
weight: 60
---

# modfloor {#modfloor}

The Go toolchain floor a module **declares**, against the floor a fix **requires**.
[Repository →](https://github.com/go-fleettools/modfloor) Latest tag: **v0.2.0**;
this page is written from its README at that tag.

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

The README names its siblings: `vulnscan` and `vulngate` for the advisories
themselves, `gopin` for the CI pin, and `staletools` for installed binaries.
