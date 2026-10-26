---
title: "landingdiff"
weight: 90
---

# landingdiff {#landingdiff}

Which of a set of landing pages is **the odd one out**, by counting what each page
puts in the shared design system.
[Repository →](https://github.com/go-fleettools/landingdiff) Latest tag: **v0.2.0**;
this page is written from its README at that tag. The README gives no install line;
the module is `github.com/go-fleettools/landingdiff`, with its `main` at the root.

```sh
landingdiff ../landing-go-*/layouts/index.html
```

```text
                               go-authn go-fileshare  go-pkgx go-pdfkit   spread
headline line 1, characters          14           10       19        23   10..23
items in the trust band               3            3        3         4   3..4  <- go-pdfkit
buttons in the hero                   2            2        2         2
...
go-pdfkit is alone on 2 measure(s)
```

(An excerpt of the README's example.)

## Why counting, and not diffing {#why}

The landings share one `styles.html`, so "aligned" cannot be checked by diffing the
design system. What drifts is **what each page puts in it**: a band with one more
item than fits on a line, a navigation with twice as many links as its neighbours.

## What it will not tell you {#limits}

- **A page is named only when the others agree and it does not**: one value held by
  everybody else, one page holding something different. It deliberately does not flag
  "furthest from the rest", which would name the smallest and the largest every time.
- **It is blind to layout.** A headline too long for its column is a fact about
  rendering, found by screenshotting the pages; use both, because neither finds the
  other's.
- **Read the comment on the line before you "fix" an outlier**: an outlier the history
  explains is not drift.

To add a measure, add one entry in `measures`: a name and a function from the page's
HTML to a count, returning **-1**, not 0, when the part is not there at all.
