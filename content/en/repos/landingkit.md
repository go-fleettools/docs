---
title: "landingkit"
weight: 20
---

# landingkit {#landingkit}

The go-* landing pages' shared design, as a **Hugo module**, and the command that
derives a landing's brand from its logo and checks a landing against the reference.
[Repository →](https://github.com/go-fleettools/landingkit) Latest tag:
**v0.12.0**; this page is written from its README at that tag. The landing of this
very organisation uses it.

Before the kit, every landing copied the partials and the copies drifted. A landing
that imports the module cannot drift; it can only fall behind a version, and Renovate
proposes the bump.

## Importing it {#import}

```toml
# hugo.toml
[module]
  [[module.imports]]
    path = "github.com/go-fleettools/landingkit"
```

```text
# go.mod, at the root of the site: Hugo resolves modules with the go command
module github.com/<org>/<org>.github.io

go 1.27.3

require github.com/go-fleettools/landingkit v0.12.0
```

Add a `kit.go` next to it, behind a `tools` build tag, importing the module: without
it `go mod tidy` finds nothing that uses the module and deletes the requirement. The
Pages workflow needs `actions/setup-go` before `hugo`. A site's own
`layouts/partials/<name>` still wins over the module's. The README has the exact
steps.

## What is in the module {#parts}

The partials include the whole design (`styles.html`, brand colours from
`[params.brand]`), the theme toggle (system, light, dark), the fork ribbon, the
Octicons icons, a loading carousel, a hero carousel, the mantra strip, an opt-in red
`btn-try` button, and a language selector with the words the partials write in seven
languages. The README documents each, with the version that added it.

## `landingkit brand` {#brand}

```sh
landingkit brand static/img/logo.svg >> hugo.toml   # then remove the old block
```

The brand colour is **read from the logo's gradient**, never chosen, and it is the
only thing an organisation sets: the first two `stop-color` of the SVG give a dark
stop (`base`, which carries text on white) and a light one (`bright`, the accent in
dark mode), and every other value is derived. The block also records both contrast
ratios, so a logo too light to carry text is visible in the diff that introduces it.
Backgrounds, surfaces and greys belong to the sheet and cannot be configured.

## `landingkit verify` {#verify}

```sh
landingkit verify [-hugo "pkgx hugo@0.167.0"] [-chrome PATH] [-pixels=false] [-out DIR] <site> <ref-site>
```

It builds both sites and fails unless **the text** holds (the built `<style>` block
equals the reference's, once brand values are masked) and **the pixels** hold
(headless Chrome renders both pages, light and dark, and the page background is
sampled where nothing but the background is painted). Builds and screenshots go to a
new temporary directory, refused inside a git work tree.

## `landingkit links` {#links}

```sh
landingkit links [-attempts 3] [-wait 3s] [-timeout 25s] public
```

Every `http(s)` URL in the **built** site must answer 2xx or 3xx, with redirects
followed, each URL tried three times. A github.com blob link is checked on the raw
host, since github.com answers a program with 503 most of the time. `workflows/links.yml`
is a template for a landing's link check.

## What it does not see {#limits}

`verify` looks only at the `<style>` block and the background: a landing whose
`index.html` differs passes. That is what [landingdiff]({{< relref "landingdiff.md" >}})
counts; use both. A landing should say what a module is for, never a count copied from
elsewhere, since that can only go stale.
