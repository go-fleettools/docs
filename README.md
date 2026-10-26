# docs

Documentation of [go-fleettools](https://github.com/go-fleettools), built with Hugo
and the [tannevaled/hextra](https://github.com/tannevaled/hextra) fork (a submodule
pinned at the release tag `v0.19.0`).

Published at <https://go-fleettools.github.io/docs/>: an overview and one page per
repository of the organisation, each written from the repository's README at its
latest tag (the tag is named on the page; a repository with no tag says so).

```sh
git clone --recurse-submodules https://github.com/go-fleettools/docs
hugo server
```

Build as CI does: `hugo --gc --minify --panicOnWarning --printPathWarnings`.
