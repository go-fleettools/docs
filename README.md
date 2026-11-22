# docs

Documentation of [go-fleettools](https://github.com/go-fleettools), built with Hugo
and the [tannevaled/hextra](https://github.com/tannevaled/hextra) fork (a submodule
pinned at the release tag `v0.19.0`).

Published at <https://go-fleettools.github.io/docs/>: an overview and one page per
repository of the organisation, each written from the repository's README at its
latest tag (the tag is named on the page; a repository with no tag says so).

Whether those tags are still the latest ones is a different question, asked
**daily** by the `labels` workflow and not on a change: a page goes stale when
*another* repository publishes a tag, so a `push` or `pull_request` trigger would
paint a contributor's branch red for somebody else's release. It is report-only
and needs no secret — `tagcells` reads each latest release with `git ls-remote`.
To fix what it reports:

```sh
tagcells --org go-fleettools --write content/en/repos/*.md
```

**Read that diff before committing it.** It replaces the version *number* and
nothing else, so a line that also names the tag's commit, or the command that
read it, comes out of the rewrite stating the new number beside the old commit —
and a page citing a commit reads as more sourced than one citing none. That is
what happened to `fleettools.md:12` here; the commit and the command were
corrected by hand.

It is told how many claims these pages hold (`min-cells: 11`), because the
refusal it would otherwise make is *"did I read none at all?"*, which stays green
while ten of eleven are read. Adding or removing a repository page is one line in
that workflow, in the pull request that adds or removes it.

```sh
git clone --recurse-submodules https://github.com/go-fleettools/docs
hugo server
```

Build as CI does: `hugo --gc --minify --panicOnWarning --printPathWarnings`.
