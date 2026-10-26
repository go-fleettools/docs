---
title: "staletools"
weight: 50
---

# staletools {#staletools}

Which installed Go tools were built with an older toolchain, and, of those, **which
actually parse Go source**, because only for those is staleness a wrong answer rather
than an old binary. [Repository →](https://github.com/go-fleettools/staletools)
Latest tag: **v0.3.0**; this page is written from its README at that tag.

```console
$ staletools
READ GO SOURCE and are behind go1.27.1 — these do not fail, they MISREPORT:
  gopls                  go1.26.4     ~/go/bin
                         go install golang.org/x/tools/gopls@latest
  ...

INSTALLED TWICE with different toolchains — PATH order decides which runs:
  agentsync
      go1.27.1     ~/.local/bin
      go1.26.4     ~/go/bin

81 Go tool(s), 29 behind go1.27.1, 8 of those read Go source
```

(An excerpt of the README's example, measured on its author's machine.)

| Exit | |
| --- | --- |
| 0 | no source-reading tool is behind, and nothing is shadowed |
| 1 | something is behind or shadowed |
| 2 | refused: a directory that cannot be read |

## Install and usage {#install}

```sh
go install github.com/go-fleettools/staletools@latest
```

```text
staletools [-all] [-want go1.27.1] [dir...]
```

With no directory it looks at `$GOBIN` (or `$GOPATH/bin`, or `~/go/bin`) and
`~/.local/bin`. `-want` compares against another toolchain: "what would be stale if
we moved to this?". It has no dependency beyond the standard library.

## Why the distinction {#why}

A CLI built with go1.26 runs under a go1.27 toolchain: it is old, not broken. But a
tool that analyses Go source embeds `go/types` and `go/parser` from its build
toolchain, and built with go1.26 it cannot read go1.27: it emits type errors about
the standard library and then reports on whatever it managed to load, so the output
reads like findings. staletools finds those tools from the **dependency list** of the
binary (`golang.org/x/tools`, `honnef.co/go/tools` and the like), not from a list of
names. It also reports a tool installed in two directories with different toolchains.
The totals are always printed, zeros included.

## `-releases` {#releases}

```sh
staletools -releases
```

The default axis asks what a tool was **built with**; this one asks what it **is**:
which installed tools are an older release of themselves. A marker `→` flags those in
a directory the shell searches, ahead of those in one nothing reads. The oracle is
`go list -m -versions`, not a tag listing, and a tool built from an unreleased commit
is reported under its own heading rather than mislabelled as behind.
