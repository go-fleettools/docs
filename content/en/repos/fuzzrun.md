---
title: "fuzzrun"
weight: 100
---

# fuzzrun {#fuzzrun}

Run every Go fuzz target of a module, in parallel within a load budget, and report
crashers. [Repository →](https://github.com/go-fleettools/fuzzrun) Latest tag:
**v0.6.0** (`go list -m -versions`, 2026-11-19). This page describes `run` and
`wait` from the README at **v0.1.0** and `audit` from the README at **v0.6.0**,
and the versions are stated separately on purpose: *which tag is current* and
*which tag this page describes* are different facts. ⚠ The repository also has
`census`, `workflow` and `adopt` subcommands, which this page does not describe
yet — they are named here so their absence reads as a gap and not as a tool that
lacks them.

`go test -fuzz` runs one target of one package at a time, so a module with thirty
targets needs thirty commands, and a run left unattended either overloads the machine
or leaves it idle. `fuzzrun` finds the targets, runs a few at once, waits for the
machine to be quiet enough before starting each one, keeps one log per target, and
says in a summary which target crashed and where the corpus file that reproduces it
was written.

## Install {#install}

```sh
GOBIN=~/.local/bin go install github.com/go-fleettools/fuzzrun@latest
```

## `fuzzrun run` {#run}

```sh
fuzzrun run [--dir .] [--time 10m] [--jobs 3] [--parallel 2] \
            [--max-load 0] [--logs DIR] [--package pattern...]
```

- Targets are found by **parsing** the `*_test.go` files (`go/parser`, not a regular
  expression): top-level `func FuzzXxx(f *testing.F)`. `.git` and `testdata` are
  skipped, and a test file that does not parse is an error, never silently zero
  targets.
- Each target runs as `go test -run=^$ -fuzz=^Name$ -fuzztime=D -parallel=P ./pkg`
  with `GOWORK=off`, at most `--jobs` at a time, its output in
  `<logs>/<pkg>-<Name>.log`.
- `--max-load L` (L > 0): before starting each target, wait while the 1-minute load
  average is above L. It is read from `sysctl vm.loadavg` on darwin and
  `/proc/loadavg` on linux. Where it cannot be read, `run` exits 2 up front instead
  of pretending the machine was quiet.
- `--package` keeps only matching packages (repeatable).
- A target that fails with a `Failing input written to testdata/fuzz/<Name>/<hash>`
  line is a `CRASH`, and the summary gives that path relative to the module. Any
  other failing exit is a `FAIL`, with the log to read.

```text
PASS  ./. FuzzGood 5s
CRASH ./. FuzzBad  1.2s testdata/fuzz/FuzzBad/6f1c...
2 targets: 1 PASS, 0 FAIL, 1 CRASH
```

Exit status: `0` every target passed, `1` at least one `FAIL` or `CRASH`, `2` the run
could not be made (no `go.mod`, no targets, `go` missing, bad flags, unreadable load
with `--max-load`).

## `fuzzrun audit` {#audit}

```sh
fuzzrun audit [--dir .]
```

Finds the fuzz targets no workflow fuzzes. It parses the test files for the
`FuzzXxx` targets, as `run` does, reads `.github/workflows/*.yml`, and prints a
`GAP` per target no step fuzzes and a `BLANKET` per fuzz step under
`continue-on-error: true` — a step that cannot fail cannot say that nothing was
fuzzed. A step running `fuzzrun run` covers every target, or those of its
`--package`. Otherwise only a `go test -fuzz` that go test would really run
counts: the pattern is an unanchored regexp, a pattern matching more than one
target of the package fuzzes none, and `-fuzz` over several packages is refused
by the go tool; those steps are `NOTE`s and their targets stay gaps.

**A static `strategy.matrix` is expanded anywhere in the step's script** — the
`-fuzz` pattern *and* the package operand — because a matrix lane names them
together: `-fuzz '^${{ matrix.target }}$' ${{ matrix.pkg }}`. Up to v0.5.0 only
the pattern was expanded, so the package stayed literal, the step read as
computed at run time, and every target it covers came out `UNCERTAIN`. Measured
on go-crdt/crdt, whose fuzz lane is twenty static `include` entries:

```text
v0.5.0   35 targets, 0 without a workflow, 35 uncertain, exit 0
v0.6.0   35 targets, 0 without a workflow, 15 uncertain, exit 0
```

The 15 that remain are a lane whose pattern a shell loop builds from a `grep`,
which no reader of the file can decide (go-crdt/crdt#184 asks that repository
whether to make it readable).

⛔ **The combinations are paired, not crossed.** The list keys give the cross
product, `exclude` removes what it matches, and each `include` entry is a job of
its own, so twenty entries pairing a package with a target stay twenty jobs
rather than becoming sixty. Coverage is keyed by *(package, target)*, and an
invented job would mark a target covered whenever the same target name exists in
another package — the one answer an audit must never produce. A matrix holding
**both** a list and `include` is not a pairing the audit reads: the two are
crossed and a `NOTE` says so.

A step the audit cannot read — a pattern the shell computes, `make fuzz`, a
script, a `working-directory` — makes the uncovered targets `UNCERTAIN` instead
of `GAP`, and they do not fail the audit.

Exit status: `0` everything covered or uncertain, `1` gaps or a blanket
`continue-on-error`, `2` nothing could be read. ⚠ Note what `0` means when
*every* target is uncertain: nothing was decided, and the summary's
`0 without a workflow` is true and empty. go-fleettools/fuzzrun#14 raised
whether that case should exit 2, as `wfaudit` and `redscan` do for a scan that
read nothing; v0.6.0 did not change it.

## `fuzzrun wait` {#wait}

```sh
fuzzrun wait FILE SUBSTRING [--timeout 10m] [--interval 2s]
```

Blocks until FILE contains SUBSTRING, for example a log written by a background
`fuzzrun run`. It prints `found` (exit 0) or `timeout` (exit 1); exit 2 on a read
error other than the file not existing yet.

## Why {#why}

The fuzz targets of a module are a security control only if they are actually run,
all of them, for long enough, without taking the machine down. The tool came out of
the go-llmflow/gateway security audit, where thirty targets were run by hand with a
shell loop that could not tell a crash from a timeout. Its tests plant a failing
target and require `run` to report it with its corpus path and exit 1; the module is
copied to a temporary directory first, so the corpus a crash writes never lands in
the repository.
