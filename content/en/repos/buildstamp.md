---
title: "buildstamp"
weight: 110
---

# buildstamp {#buildstamp}

Read the Go build information of any binary, WebAssembly included, and fail one built
from a modified tree. [Repository →](https://github.com/go-fleettools/buildstamp)
Latest tag: **v0.1.0**; this page is written from the facts of that release.

`go version -m` and `debug/buildinfo` answer "unrecognized file format" on a `.wasm`
file. The go command writes the module information into every binary it links, between
two 16-byte sentinels (`infoStart` and `infoEnd` in `cmd/go/internal/modload/build.go`).
`buildstamp` finds them and parses the text with `runtime/debug.ParseBuildInfo`, so the
same answer comes out of an ELF, a Mach-O, a PE or a WebAssembly module.

## Install {#install}

```sh
GOBIN=~/.local/bin go install github.com/go-fleettools/buildstamp/cmd/buildstamp@v0.1.0
```

## `buildstamp check` {#check}

```sh
buildstamp check [--revision SHA] FILE...
```

A file is clean when its build was stamped from an unmodified tree (`vcs.modified` is
`false`) and, with `--revision`, from that commit. Every file is reported, even after
the first failure.

Exit status: `0` every file is clean, `1` at least one is not (modified tree, no vcs
stamp, or another revision), `2` a file is unreadable or carries no Go build
information.

In a workflow, build outside the checkout and check against the commit under test:

```sh
go run github.com/go-fleettools/buildstamp/cmd/buildstamp@v0.1.0 \
  check --revision "$GITHUB_SHA" "$RUNNER_TEMP/site/app.wasm"
```

## `buildstamp show` {#show}

```sh
buildstamp show FILE...
```

Prints the build information of each file: toolchain, module path and version, and the
build settings.

## Package {#package}

`github.com/go-fleettools/buildstamp` exposes `ReadFile`, `Read`, `Setting` and
`Clean(bi, wantRevision)` for programs that need the same answer.

## Why {#why}

A CI build made after something wrote into the checkout is stamped
`vcs.modified=true` and version `v0.0.0-…+dirty`. `go-wasmcontainers/dashboard` v0.4.2
shipped that way: its Pages job wrote `_site/` inside the work tree before building.

## Safety {#safety}

Reads are bounded: 1 MiB of module information, and 1 GiB per file, checked by `Stat`
before reading. `FuzzRead` runs in CI and the package is covered at 100% of
statements. The tests build real `js/wasm` binaries, clean and dirty, and a CI lane runs
the tool on its own build and on a planted dirty build that must exit 1.
