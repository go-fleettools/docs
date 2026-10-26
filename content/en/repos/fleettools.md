---
title: "fleettools"
weight: 10
---

# fleettools {#fleettools}

Sweeps across about 1800 repositories and about 320 organisations: which default
branches are red, which runners stopped, which modules are ahead of their tag, which
foreign judges the CI never installs, and which tests exist without ever running.
[Repository →](https://github.com/go-fleettools/fleettools) Latest tag:
**v0.40.0**; this page is written from the top of its README at that tag, which is
long (about 430 lines), and from the layout of the repository. Read the README for
each command.

The sweeps were written to get every default branch green, and the README says why
each one is built the way it is. It has **one directory per sweep under `cmd/`**; the
module is `github.com/go-fleettools/fleettools`, and a command is installed as Go
installs tools (the README's example is `go install .../cmd/vulngate@v0.18.1`).

## The commands {#commands}

The directories under `cmd/` at v0.40.0 are:

```text
abbench blobpurge brandcheck brief buildscan buildtag clonescan covblocks cronscan
docscan docscheck docsversions docsym glmerge gopin gorequires gvlscan hostload
i18ncheck judgescan landingmigrate langcheck mantracheck mkdocs2hextra modrename
pagescheck pagespublish pagesretire pagetext pinscan pngeq prmerge prscan qemufix
quietscan redscan relverify sitelinks tagcells tagscan testscan treecmp truncspill
unarc vscan vulngate vulnscan wasmsize wfaudit
```

The README's table, which opens with the sweeps written on 2026-08-30, describes
some of them, for example:

| Command | What it does |
| --- | --- |
| `redscan` | every default branch whose last run failed. A red branch is not a broken one: check **when** it last ran |
| `prscan` | open pull requests across every organisation, batching organisations so that about 20 search calls replace 320 |
| `prmerge` | merges dependency pull requests that are mergeable **and** fully green |
| `tagscan` | every tagged Go module whose default branch is **ahead of its latest tag**, work that is merged and that `go get` cannot reach. It asks about tags, not releases |

Others are mentioned by their sibling repositories, for instance `vulnscan` (runs
govulncheck across the fleet), `vulngate` (judges one run) and `gopin` (moves the CI
pin) in the [modfloor]({{< relref "modfloor.md" >}}) page, and `pagespublish`, which
publishes a documentation version in the workflows of every docs site, this one
included.

## What is built into them {#built-in}

- **They say when they are not the code in this repository** (a binary installed
  from an old tag is compared with what changed for that command).
- **Every one answers `-h` with 0.** `redscan -h` once ran a full sweep of every
  organisation, and a test now fails any command that cannot answer `-h`.
- **A sweep asks for the hourly API budget before spending it** and refuses to begin
  below a floor, because that budget is per user and shared with the fleet's Renovate
  runners.
- **A failed call must never become a verdict**, and a budget stop is reported
  separately from "could not be read".

## `quietscan` {#quietscan}

The README devotes sections to `quietscan`: "What `quietscan` is, and the five rules
it will not soften", attribution ("the property belongs to whatever COVERS the
repository"), "A failed call must never become a verdict", "What one pass costs, and
when not to run it", and "Two things `quietscan` is not". Read them there before
scheduling it beside another sweep.
