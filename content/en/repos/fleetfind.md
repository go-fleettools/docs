---
title: "fleetfind"
weight: 80
---

# fleetfind {#fleetfind}

**Which repository already does this?** A live, dated inventory of every
organisation, because a hand-kept capability map drifts.
[Repository →](https://github.com/go-fleettools/fleetfind) Latest tag: **v0.4.0**
(`git ls-remote --tags`, 2026-11-15); this page is written from its README at
**v0.4.0**.

```sh
go install github.com/go-fleettools/fleetfind@v0.4.0
```

```sh
fleetfind -refresh             # ~377 gh calls: one dated inventory, then the file/function index
fleetfind -index               # only the index, from the module proxy: 0 GitHub calls
fleetfind odf rtf markdown
```

```text
go-odf/odf                         ODT (OpenDocument Text) ⇄ richdoc converter, pure Go (CGO-free)
go-rtf/rtf                         RTF <-> richdoc converter, pure Go (CGO-free)
go-richdoc/markdown                Markdown (CommonMark + GFM) <-> richdoc converter

3 of 2023 repositories, inventory of 2026-10-08
```

(The README's example, with the date of its inventory.)

## Files and functions, not only domains {#index}

A repository's name and description name its domain, never its functions, so
`fleetfind quantile` used to print `nothing in 2253 repositories` while
`go-statistics/stats` has `quantile.go`. The index adds file names and the exported
top-level identifiers of every Go module, read from the Go module proxy (0 GitHub
calls, incremental on `pushed_at`, bounded and zip-slip safe):

```text
go-statistics/stats   file:quantile.go
go-statistics/stats   ident:Quantile
```

Every run says how many repositories were indexed, how many are not in the proxy
(not a module, **private**, or unpublished: a blind spot, not a NO), and how many
are **UNREAD** with the reason. A zero is never printed as `nothing in N
repositories` when part of N could not be read.

## What it will not let you believe {#rules}

- **Every answer carries the date the inventory was taken**, and one older than a day
  says so on stderr before it answers: a "nothing matches" is only as good as when it
  was measured.
- **Zero results are said out loud**, because a sweep that prints nothing is
  indistinguishable from one that could not read.
- **An empty fleet is refused**, twice and separately (no organisations listed;
  organisations that listed and brought back no repository), and neither writes an
  empty inventory over a good one.
- **Archived repositories are found but not returned by default**; `-all` asks for
  them, because knowing something moved is the useful answer.

## Ranking {#ranking}

A name beats a description: a path match scores 10 and a description or topic match
3. Every term must match, so two words narrow, they do not widen.

A hit is a lead, not an authority. Confirm it:

```sh
gh api repos/<org>/<repo> --jq '{full_name, id, pushed_at, archived}'
```
