---
title: "mutate"
weight: 30
---

# mutate {#mutate}

**Prove that a test can fail.** Apply one exact source mutation, run the suite,
restore whatever happens, and exit 0 only when the suite went **red**.
[Repository →](https://github.com/go-fleettools/mutate) Latest tag: **v0.7.0**; this
page is written from its README at that tag. It has no dependency beyond the standard
library.

```console
$ mutate -file browse.go -name "mark repeats too" \
    -from 'if !n.Repeat {' -to 'if true {' \
    -- go test -run TestARepeatIsNot ./...
mutate: mark repeats too — caught
```

## Install {#install}

```sh
go install github.com/go-fleettools/mutate@latest          # one mutation
go install github.com/go-fleettools/mutate/mutsweep@latest # every refusal in a package
```

## `mutsweep`: the same question, of a whole package {#mutsweep}

`mutate` proves one test can fail. `mutsweep` asks it of every refusal in a package:
it deletes each one in turn, and says which ones nothing noticed.

```console
$ mutsweep -dir structured -files blob.go,cell.go -run-in . -- go test ./...
mutsweep: 38 refusals, 3m0s each at most, through "go test ./..."
[1/38] caught       blob.go:366  (size == 0) != (count == 0)  5s
[15/38] SURVIVED    blob.go:347  used <= 0  12s
...

38 refusals: 15 caught, 21 not mutants, 2 survived, 0 hung

what nothing holds:
  SURVIVED   blob.go:347  used <= 0
  SURVIVED   cell.go:98   !ok
```

The README reports that over five runs on one project it put 574 guards through this
and found thirteen that no test held.

### Answering a report: `-only` {#only}

After a sweep, tests are written for the survivors, and the question becomes whether
they *kill* them, which is not the same as whether they pass. `-only` re-asks it for
the named targets, spelled as the report prints them:

```sh
mutsweep -only map.go:163,map.go:628 -timeout 2m -- go test -count 1 ./...
```

A target that matches no refusal is an error naming it, because an `-only` list goes
stale and a sweep of nothing reads like a clean one.

### A file the build excludes {#excluded}

A `//go:build js && wasm` file is not in a native build, so deleting a refusal in it
changes nothing the command can see: the verdict would be SURVIVED when nothing
looked. `mutsweep` reads `GOOS` and `GOARCH` as the go tool does. Naming an excluded
file in `-files` or `-only` is an error (exit 2), and a directory sweep skips such
files and **names** them.

The README also describes `-tally`, for planning and auditing a campaign swept in
chunks; read it there.
