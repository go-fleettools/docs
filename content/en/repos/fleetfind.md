---
title: "fleetfind"
weight: 80
---

# fleetfind {#fleetfind}

**Which repository already does this?** A live, dated inventory of every
organisation, because a hand-kept capability map drifts.
[Repository →](https://github.com/go-fleettools/fleetfind) Latest tag: **v0.2.0**;
this page is written from its README at that tag. The README gives no install line;
the module is `github.com/go-fleettools/fleetfind`, with its `main` at the root.

```sh
fleetfind -refresh             # ~335 calls, one dated inventory
fleetfind odf rtf markdown
```

```text
go-odf/odf                         ODT (OpenDocument Text) ⇄ richdoc converter, pure Go (CGO-free)
go-rtf/rtf                         RTF <-> richdoc converter, pure Go (CGO-free)
go-richdoc/markdown                Markdown (CommonMark + GFM) <-> richdoc converter

3 of 2023 repositories, inventory of 2026-10-08
```

(The README's example, with the date of its inventory.)

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
