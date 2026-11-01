---
title: "fleettools"
weight: 10
---

# fleettools {#fleettools}

Sweeps across about 1800 repositories and about 320 organisations: which default
branches are red, which runners stopped or run and fail, which modules are ahead of
their tag, which foreign judges the CI never installs, and which tests exist without
ever running. [Repository →](https://github.com/go-fleettools/fleettools) Latest tag:
**v0.67.0**; this page is written from the repository at that tag — the command
inventory under `cmd/`, the README's own sections, and each new command's doc comment.
The README is about 610 lines and describes every command in its own words; read it
there.

The sweeps were written on 2026-08-30 while getting every default branch green, and
the README says why each one is built the way it is. The repository has **one
directory per sweep under `cmd/`**; the module is
`github.com/go-fleettools/fleettools`, and a command is installed the way Go installs
tools, at a tag (`go install github.com/go-fleettools/fleettools/cmd/vulngate@v0.67.0`).

## The commands {#commands}

**55 directories under `cmd/`** at v0.67.0, each with a `main.go` (counted
2026-11-01):

```text
abbench blobpurge brandcheck brief buildscan buildtag clonescan covblocks cronscan
docscan docscheck docsversions docsym flagcensus glmerge gofloor gopin gorequires
gvlscan hostload i18ncheck judgescan landingmigrate langcheck mantracheck mdpdf
mkdocs2hextra modrename needread pagescheck pagespublish pagesretire pagetext pinscan
pngeq prmerge prscan qemufix quietscan redscan relverify sitelinks sitemigrate symfind
tagcells tagscan testscan treecmp truncspill unarc vscan vulngate vulnscan wasmsize
wfaudit
```

Grouped by the question they answer:

- **Is the fleet green?** `redscan` (every default branch whose last run failed — a red
  branch is not a broken one, check *when* it last ran), `quietscan`, `prscan`,
  `prmerge`, `glmerge` (the GitLab side), `relverify`, `cronscan`.
- **Is what is merged reachable, and is the version right?** `tagscan` (a module whose
  branch is ahead of its latest tag: work that is merged and `go get` cannot reach — it
  asks about **tags**, not releases), `vscan` (what the next version should be, derived
  from the public API with `gorelease`, once per GOOS), `tagcells`, `gorequires`,
  `pinscan`, `modrename`.
- **Does the CI actually test it?** `testscan`, `judgescan` (tests that look for an
  external tool the CI never installs — a foreign judge that exists in the source and
  has never once run), `covblocks` (the blocks a coverage profile never entered, because
  `go tool cover -func` rounds to one decimal and a 100 % gate passes at 99.95 %),
  `buildscan`, `buildtag`, `flagcensus`.
- **Is it safe?** `vulnscan`, `vulngate` (the per-repository half), `gofloor` (which Go
  toolchain a repository can be built with, and whether CI would notice a vulnerable
  one), `wfaudit`, `blobpurge`, `needread` (the `DT_NEEDED` entries of every ELF under a
  tree, refusing to report unless a symbol it was told to expect is actually there),
  `symfind` (whether an ELF *exports* a dynamic symbol, rather than merely consuming it).
- **Is it advertised, and is the page what it builds?** `docscan`, `docscheck` (which
  generator a published URL was built with), `pagescheck` (whether a Pages site serves
  what its repository builds), `pagetext` (whether a published page now carries the text
  we wrote), `sitelinks` (whether every local link of a built site leads to a file of
  that build), `docsym`, `i18ncheck`, `langcheck`, `mantracheck`, `brandcheck`.
- **Publishing the documentation.** `pagespublish`, `pagesretire`, `mkdocs2hextra`,
  `sitemigrate`, `landingmigrate`, `docsversions` (every published version of a
  versioned site built with **one** look — the default branch's), `mdpdf`.
- **What does it cost, and on what?** `wasmsize`, `hostload`, `abbench`, `truncspill`,
  `gvlscan`, `gopin`, `qemufix`.
- **Comparing artefacts.** `treecmp` (byte for byte), `pngeq` (the same *pixels*, because
  "65 of 73 files differ" on a moved brand kit was false — 51 PNGs, 0 channel values
  different).
- **The small ones.** `brief` (runs a command, shows the end of what it said, and keeps
  its exit status), `unarc`, `truncspill`, `clonescan`.

## What is built into them {#built-in}

- **They say when they are not the code in this repository.** A binary installed from
  an old tag is compared with what has changed for that command since.
- **Every one answers `-h` with 0.** `redscan -h` once ran a full sweep of every
  organisation and had to be killed; a test now fails any command that cannot answer
  `-h`.
- **A sweep asks for the hourly API budget before spending it** and refuses to begin
  below a floor, because that budget is **per user** and shared with the fleet's
  Renovate runners. Four sweeps have emptied it and made those runs fail.
- **A failed call must never become a verdict**, and a deliberate budget stop is
  reported separately from "could not be read": an unread repository is asked again, a
  budget stop is waited out.
- **The secondary rate limit is not visible to `rate_limit`** — measured 2026-10-10,
  `gh` answered 403 on every call while the budget read 5000/5000.

⛔ **And two of them were blind to this fleet's own pages until v0.67.0**, which is
the same failure one level up: a tool that is right about its rules and cannot read
the subject answers confidently anyway.

- `tagcells` keeps the version a page states equal to each module's latest release.
  It looked for `[[params.modules]]` with `tag = "vX.Y.Z"`; the go-* landings are
  `[[params.repos]]` with `label =`. Against `go-fleettools.github.io/hugo.toml` it
  answered `0 cell(s)` and exited **2** — rightly refusing to call nothing a pass, and
  the reason thirteen labels dated in a comment above them had never been checked.
  When it first could look, **three of the thirteen were stale** and `fleettools`'
  own was twenty-seven releases behind.
- `pagetext` reported text **missing** on a page that carried it, six retries running,
  because of an apostrophe: Hugo prints `fleet’s` where the source said `fleet's`, and
  the match decoded entities but not typographic punctuation. Every English possessive
  and contraction was a false negative. A verification tool that says "not published
  yet" about something published is worse than none — the next person reads a real
  failure the same way.

## Flags: one dash became two {#flags}

Since **v0.53.0** all of the commands use [cobra](https://github.com/spf13/cobra), so
the canonical spelling is `--long` with POSIX short forms. The fleet had typed
`-dir`, `-apply`, `-org` for months, and pflag reads one dash as **one letter**, so
every one of those would have been refused: a single-dash compatibility shim in
`internal/cli` rewrites the old spelling and warns, which is why there was no flag day.
`cmd/flagcensus` is the measurement that drove it — 54 commands, 144 flags, 137 of
which needed the second dash.

## `quietscan` {#quietscan}

`quietscan` is every organisation whose Renovate runner has **stopped**, *or* **runs
and fails**. `redscan` asks whether the last run failed; a runner that stopped does not
fail, it says nothing — so this asks *when* it last ran, and then what became of it.
**Sixteen runners were once reported healthy while every nightly run failed in ten
seconds**, which is the second half the README added.

The README devotes sections to it: "What `quietscan` is, and the **six** rules it will
not soften", attribution ("the property belongs to whatever COVERS the repository"),
"The trap `quietscan` exists to catch before it latches", "A failed call must never
become a verdict", "What one pass costs, and when not to run it", and "Two things
`quietscan` is not". Read them there before scheduling it beside another sweep.
