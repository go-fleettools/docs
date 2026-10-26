---
title: "unbounded"
weight: 70
---

# unbounded {#unbounded}

**Which repositories read a ZIP entry, or decode an image, without a ceiling?**
[Repository →](https://github.com/go-fleettools/unbounded) Latest tag: **v0.2.0**;
this page is written from its README at that tag. The README gives no install line;
the module is `github.com/go-fleettools/unbounded` and its command is in `cmd/unbounded`.

```sh
unbounded go-odf go-pdfkit go-gfx     # organisations: listed, cloned, parsed
unbounded ./some/checkout             # a directory is scanned in place
```

```text
  go-gfx/gfx          svg/svg.go:817  image.Decode and nothing in this file asks DecodeConfig first
  go-pdfkit/pdfkit    image.go:99     png.Decode and nothing in this file asks DecodeConfig first

66 repositories read, 0 NOT read
2 open a ZIP, 15 decode an image, 4 lead(s) to look at
```

(The README's example output.)

## Why {#why}

Both shapes are how a few hundred bytes of input become a few gigabytes of memory:
`io.ReadAll` on a ZIP entry (an `.ods` of 67,206 bytes inflated to 64 MiB and held
222.9 MiB), and `image.Decode` before `image.DecodeConfig` (a 75-byte PNG claiming
10000 x 10000 allocated 381.6 MiB). A fix does not travel: a sibling repository carried
the identical defect for weeks because nobody looked, and this is the sweep that
follows a fix across its siblings.

## A lead, not a verdict {#lead}

Both shapes are correct over input the program produced itself, so every hit has to
be read by somebody who knows where the bytes come from. What the tool owes in return:
it **parses** rather than greps; it **names what it could not read**; it prints both
denominators ("66 repositories read, 0 NOT read"); and **reading nothing is a
failure**, exiting non-zero.

A **directory** is scanned in place so that the instrument can be checked against a
checkout of a version that had the defect: it must find it, and find nothing on the
fixed version.

## What it does not cover {#limits}

The image decoders are a written-out list, so a third-party HEIC or JPEG 2000 reader
is a decoder it does not know. A ZIP entry read through a helper in another file is
missed: the two shapes are judged within one file. It holds 100 % statement coverage,
gated in CI.
