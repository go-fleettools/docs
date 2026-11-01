---
title: "Overview"
weight: 10
---

# go-fleettools documentation {#go-fleettools}

go-fleettools holds the small Go programs a fleet of hundreds of repositories
needs and nobody wants to do by hand, and the **landing kit** the go-* landing pages
share. Each tool was built after a wrong answer cost something, and each one's README
says which.

Everything is Go. The code is BSD-3-Clause: `mantracheck` found, on 2026-10-26,
13 repositories with no code under another licence and no cgo signal; the one other
licence is data (landingkit's Octicons icons, MIT). This page and the ones below are written from each
repository's README **at its latest tag, read on 2026-10-26**; the tag is named on
the page, and a repository with no tag says so. Where a README gives no install
line, the page says that too.

## The repositories {#repositories}

| Page | Tag | What it answers |
| --- | --- | --- |
| [fleettools]({{< relref "repos/fleettools.md" >}}) | v0.40.1 | sweeps across about 1800 repositories: red branches, stopped runners, modules ahead of their tag, foreign judges, tests that never run |
| [landingkit]({{< relref "repos/landingkit.md" >}}) | v0.12.0 | the landings' shared design, and the commands that derive a brand, verify a landing and check its links |
| [mutate]({{< relref "repos/mutate.md" >}}) | v0.7.0 | can this test fail? One mutation (`mutate`), or every refusal in a package (`mutsweep`) |
| [prwait]({{< relref "repos/prwait.md" >}}) | v0.5.0 | are a pull request's checks finished, without calling an empty list a pass |
| [staletools]({{< relref "repos/staletools.md" >}}) | v0.3.0 | which installed Go tools are built with an older toolchain, and which of those parse Go source |
| [modfloor]({{< relref "repos/modfloor.md" >}}) | v0.6.0 | the Go floor a module declares, against the floor a fix requires, split by whether a consumer can reach it, costed in pull requests rather than findings, with a baseline so the check is a gate and not a permanent red, and which fleet modules are required at a pseudo-version |
| [unbounded]({{< relref "repos/unbounded.md" >}}) | v0.2.0 | which repositories read a ZIP entry or decode an image without a ceiling |
| [fleetfind]({{< relref "repos/fleetfind.md" >}}) | v0.2.0 | which repository already does this |
| [landingdiff]({{< relref "repos/landingdiff.md" >}}) | v0.2.0 | which landing page is the odd one out |
| [fuzzrun]({{< relref "repos/fuzzrun.md" >}}) | v0.1.0 | run every fuzz target of a module within a load budget |
| [buildstamp]({{< relref "repos/buildstamp.md" >}}) | v0.1.0 | the Go build information of any binary, WebAssembly included, and whether it was built from a modified tree |
| [.github]({{< relref "repos/dotgithub.md" >}}) | no tag | shared community health files, Renovate runner and preset |

## What they have in common {#in-common}

- **Silence is not a pass.** prwait refuses an empty check list, mutate exits 0
  only when the suite went red, and unbounded exits non-zero when it read nothing.
- **Every answer carries its date**, as fleetfind does, and says so when it is old.
- **A lead, not a verdict**: unbounded and landingdiff say where to look and what
  they cannot see.
- **A control that could fail.** fuzzrun's tests plant a failing target, and
  landingkit's CI checks a landing with a tinted background and requires it to fail.

Links: the [organisation](https://github.com/go-fleettools) on GitHub, and the
[landing page](https://go-fleettools.github.io/).
