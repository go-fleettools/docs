---
title: "mutate"
weight: 30
---

# mutate {#mutate}

**Prove that a test can fail.** Apply one exact source mutation, run the suite,
restore whatever happens, and exit 0 only when the suite went **red**.
[Repository →](https://github.com/go-fleettools/mutate) Latest tag: **v0.14.0**
(`git ls-remote --tags`, 2026-11-22 23:49 CET, released at `7823efba` — the
merge commit of #36, CI green on `main`); this page is written from its README at
**v0.14.0**, and the two are stated separately on purpose — *which tag is current*
and *which tag this page describes* are different facts, and writing one where the
other belongs is how a page comes to assert a version nobody checked it against.
It has no dependency beyond the standard library.

```console
$ mutate -file browse.go -name "mark repeats too" \
    -from 'if !n.Repeat {' -to 'if true {' \
    -- go test -run TestARepeatIsNot ./...
mutate: mark repeats too — caught
```

## Install {#install}

```sh
go install github.com/go-fleettools/mutate@latest          # one mutation
go install github.com/go-fleettools/mutate/mutsweep@latest # every refusal in a package
```

## `mutsweep`: the same question, of a whole package {#mutsweep}

`mutate` proves one test can fail. `mutsweep` asks it of every refusal in a package:
it deletes each one in turn, and says which ones nothing noticed.

```console
$ mutsweep -dir structured -files blob.go,cell.go -run-in . -- go test ./...
mutsweep: 38 refusals, 3m0s each at most, through "go test ./..."
[1/38] caught       blob.go:366  (size == 0) != (count == 0)  5s
[15/38] SURVIVED    blob.go:347  used <= 0  12s
...

38 refusals: 15 caught, 21 not mutants, 2 survived, 0 hung

what nothing holds:
  SURVIVED   blob.go:347  used <= 0
  SURVIVED   cell.go:98   !ok
```

The README reports that over five runs on one project it put 574 guards through this
and found thirteen that no test held.

### Five verdicts, because two is not enough {#verdicts}

| verdict | meaning |
| --- | --- |
| **caught** | the suite went red and a test printed an assertion line: the guard is held |
| **caught by PANIC** | the suite went red, the output holds a runtime panic and no test asserted: held by luck, and **listed with the survivors** |
| **SURVIVED** | the suite stayed green: nothing holds it |
| **not a mutant** | neither mutation compiled — the deletion, then `false && (C)` |
| **HUNG** | no answer in time: the guard is load-bearing *and* nothing says so |

The exit status is **non-zero** when anything survived, hung, or was caught only by
a panic, so a sweep can stand as a CI gate. `HUNG` is not a theoretical category:
deleting the check that a retry policy is honourable does not fail a suite, it makes
the suite wait out an hour somebody mistyped into the wrong field.

#### A panic is not an assertion {#panic}

*"The suite went red"* has two causes, and only one of them is a test. A test that
asserts fails because it compared a value with the one it expected. A test that
merely walks past a deleted guard fails because the line after it indexed past an
array or dereferenced nil — a crash no author wrote an expectation for.

Twice in one day in `go-numerics/special` the only thing that made a mutated range
guard red was such a panic (`x < 512` written `x <= 512` failing with `index out of
range`, special#71, and `x < jyFastMaxX` the same way, special#73), and a sweep that
counts any red as caught reported both as held. **A panic red is luck**: the next
change that enlarges the table makes the mutant survive, silently. So a red suite is
read once more, from `go test -json` events, and a panic-only red is given its own
verdict and listed among the things nothing holds.

⛔ **And that verdict used to move with the wording of a message.** A panic was any
line containing `panic:` followed, at any distance, by the runtime's `goroutine N
[state]:` header. So a decline test that reports a *recovered* crash — which is the
ordinary shape of a test written to hold a nil check —

```go
t.Errorf("empty: %v, want an error wrapping ErrMalformed", err) // err wraps "panic: runtime error: …"
```

tainted **its own** test, its assertion lines were discarded, and a guard held *by an
assertion* was certified `caught by PANIC`: the verdict whose meaning is *this red is
luck*. Measured on `go-pdfkit/reader` `signature.go:69`: rewording that helper from
`panic: %v` to `crashed (recovered): %v`, with no change to what the test asserts,
turned the same sweep into `caught`. ⭐ A verdict that moves with the wording of a
message is not a measurement.

The rule now: **an assertion line is never the start of a panic.** The runtime prints
`panic: …` at **column 0**; `t.Error` and `t.Fatal` print an *indented* source
position, so the line's own prefix states who printed it, and *"ambiguity resolves to
PANIC"* has no ambiguity to resolve. The control on the other side is a fixture whose
panic *value* prints an assertion-looking line: its `panic:` sits at column 0 on the
line before, so it is still a panic.

#### A verdict line with no name is not a verdict {#nameless}

⛔ **A real panic read as `caught` — because of a line testify prints.** The reader
that turns go test's plain output into verdicts had an arm for `--- PASS:` and
`--- SKIP:` that matched the prefix `--- ` and took the test's name from after the
colon:

```go
test = firstField(trimmed[strings.IndexByte(trimmed, ':')+1:])
passed[testKey{pkg, test}] = true
```

`IndexByte` answers **-1** when the byte is absent, and `-1+1 == 0` is a perfectly
valid index, so with no colon the slice silently became *the whole line*. Two
ordinary inputs arrive that way:

| input | what it did |
| --- | --- |
| `--- Expected` | registered a test **named `---`** as having **passed**, and filed every later line under it — the panic included. |
| `--- FAIL:` truncated | the FAIL arm needs the trailing space of `"--- FAIL: "`, so a cut line fell through here and marked a **failing** test as passed, under the *empty* name — the exact key the panic rule asks before deciding that a panic belongs to no test. |

`--- Expected` is not a contrived string: it is what **every failed testify
`assert.Equal` prints**. testify sets difflib's `FromFile` to `"Expected"`
(`assert/assertions.go:1965` at v1.12.1, unchanged back to v1.3.0) and difflib
writes it as `--- %s` (`difflib.go:589`). Measured through `classify`:

```text
before: "caught"           passed = map[{p0 ---}:true {p0 TestDecode}:false]
after:  "caught by PANIC"  passed = map[{p0 TestDecode}:false]
```

Both arms now require a non-empty name, and an unrecognised `--- ` line leaves the
current test **unchanged** rather than clearing it: such a line is something a test
*printed*, so what follows still belongs to that test — which is also the safe
direction, since the lines of a test that panicked are never read as assertions.

⭐ **A prefix is not a grammar, and fixtures are imagined input.** The unit suite and
the end-to-end selfcheck on three operating systems were **green** on both inputs,
because no one had thought to write them. What found it is a property, stated as an
invariant over the structure the rule later trusts — *no verdict is ever recorded
about a test with no name* — and it took **15.56 s**:

```bash
go test -run xxx -fuzz FuzzClassify ./mutsweep/
go test -run xxx -fuzz FuzzReadersKeepAVerdictAboutATest ./mutsweep/
```

The targets deliberately assert **no verdict**. A fuzzer has no oracle for *"is this
red a panic"*, and writing one would mean reimplementing the rule beside itself, so
the two would agree by construction and measure nothing; the fixtures that run the
real `go` tool are that half. Reduced crashers are committed under
`mutsweep/testdata/fuzz/`, so they replay in a plain `go test`.

### Answering a report: `-only` {#only}

After a sweep, tests are written for the survivors, and the question becomes whether
they *kill* them, which is not the same as whether they pass. `-only` re-asks it for
the named targets, spelled as the report prints them:

```sh
mutsweep -only map.go:163,map.go:628 -timeout 2m -- go test -count 1 ./...
```

A target that matches no refusal is an error naming it, because an `-only` list goes
stale and a sweep of nothing reads like a clean one.

### A file the build excludes {#excluded}

A `//go:build js && wasm` file is not in a native build, so deleting a refusal in it
changes nothing the command can see: the verdict would be SURVIVED when nothing
looked. `mutsweep` reads `GOOS` and `GOARCH` as the go tool does. Naming an excluded
file in `-files` or `-only` is an error (exit 2), and a directory sweep skips such
files and **names** them.

### Init-clause refusals, and a deletion that does not compile {#init}

`if v, err := f(); err != nil { return … }` is a refusal like any other. It used to be
skipped — not swept, not counted, not reported — so a sweep read as clean beside a
guard whose test passed with the guard neutralised. Deleting such an `if` would delete
the statement in its Init clause too, so it is mutated differently: the condition `C`
becomes `false && (C)`. The Init statement and every use of its variables stay, and the
body is unreachable.

The same mutation is the **fallback** for an ordinary refusal whose deletion does not
compile, because a variable used only in the condition is left unused. Before the
fallback, "not a mutant" was often **half** of everything tried — 242 of 574 — and 8 of
11 refusals in one file read that way while being guards nothing tested. "Not a mutant"
now means only that neither mutation compiles. `-mutate boundary` visits the conditions
of init-clause `if`s too.

### An `else` branch, and the `else if` nobody listed {#else}

⛔⛔ An `if` that had an `else` was **excluded outright**, and an `else if` was never
reached at all: it is **not an element of any statement list** — it lives in the `Else`
field of the `if` before it — so a walk over statement lists sees its *body* and never
its *condition*. Two different reasons, one symptom: not swept, not counted, not
reported, so a package of `if`/`else` guards read as a **clean sweep about a shape the
walk never looked at**.

| shape | before | now |
|---|---|---|
| `if x > 0 { … }` | listed | listed |
| `if v, ok := m[k]; ok { … }` | listed | listed |
| `if x > 10 { … } else if x > 5 { … }` | **neither condition** | both |
| `if x > 3 { … } else { … }` | **nothing** | listed |
| the same two branches with no `else` | listed | listed |

⭐ **The last two rows are the control**: the same condition, the same two branches,
differing only by the `else`, and only one of them was swept. Without that pair, *"an
`if` with an `else` is skipped"* and *"an `else if` condition is skipped"* are two
readings of one symptom — and the first account of the finding named **neither**,
blaming the init clause the section above had already fixed. The control was built
before the tool was touched, which is the only reason that guess did not become the
patch.

Deletion is the wrong mutation here, which is most likely why the exclusion was
written: deleting `if C { A } else { B }` removes *both* arms — the decision removed
rather than the guard removed — and the suite would go red for a reason that says
nothing about `C`. So the mutation is the `false && (C)` above: every arm stays
compilable and the `else` arm runs unconditionally. It is primary when a refusal has an
Init clause, **or** an `else`, **or is itself somebody's `else`**, and the third is not
implied by the other two — the last link of a chain, `} else if x > 5 { return 1 }`,
has neither an Init nor an `else` of its own, and deleting its bytes leaves a dangling
`} else` that does not compile. What disqualifies deletion is **being** an `else`, not
having one.

#### A guard whose removal panics is a guard no test holds {#unholdable}

Extracting that per-`if` collection into its own function made its two **rejection**
sentences changed refusals, so [`-hook`](#hook) demanded a sweep of them — and both
SURVIVED. Nothing in the suite had an `if` with a long body, and nothing had one that
does not end in a `return`, so *"a refusal is a **short** body ending in a
**return**"*, the definition the whole tool rests on, was asserted **nowhere**.

⛔ One of the two could not be held by **any** assertion as it was written:

```go
if len(ifs.Body.List) == 0 || len(ifs.Body.List) > 3 { return nil }
if _, isReturn := ifs.Body.List[len(ifs.Body.List)-1].(*ast.ReturnStmt); !isReturn { … }
```

Delete the first line and the second indexes `-1`: the suite is red by **panic**, which
is the verdict this tool prints beside the survivors, because the line doing the
catching is the one below and the next change that reorders them makes the same mutant
survive in silence. There is **no test to write** for such a line — no input makes
`List[-1]` fail by assertion. The answer is to restructure until the question has an
answer a test can read: a last statement that is `nil` for an empty body, refused by
the same `*ast.ReturnStmt` test as any other non-return. Identical behaviour for every
input, one fewer guard, and both remaining bounds now caught by assertion.

### A sweep leaves a record, and `-hook` refuses a push no sweep covered {#hook}

A sweep used to leave nothing but its report, which made this an **analysis tool, not a
gate**: nothing failed when it was not run. That was measured, not argued. Over
2026-11-18/19 two sessions wrote *"break-checked"* over hand-made mutations while
`mutsweep` sat installed, and when it was finally run it found three refusals no test
held — all in code **already merged**.

So a delete-mode sweep files a **record** per file it swept completely, keyed by the
SHA-256 of the file's bytes, under the user config directory (`MUTSWEEP_RECORDS`
overrides it). And:

```sh
mutsweep -hook   # a Claude Code PreToolUse hook on Bash
```

reads the hook payload and, when the command pushes with `gitpush`, refuses the push if
it sends a non-test Go file holding refusals that:

| the refusals in the file | the push |
| --- | --- |
| have **no record at the bytes committed at HEAD** | refused: never swept as it is now |
| have a record with **SURVIVED** or **HUNG** | refused, naming the line |
| were swept by an older walk that found **fewer** refusals | refused |
| have only **caught by PANIC** | allowed, and said |

Editing a file changes its key, so a record cannot vouch for bytes it never saw — a
comment included. The way out is the loop the report recommends: write the test a
survivor asks for, re-ask that line alone with `-only file.go:line`, and the `-only` run
**updates** the record for those bytes without re-sweeping the file.

⚠ **caught by PANIC is not refused**, deliberately. For an *indexing* guard — one whose
deletion makes the next line index out of range — a panic is the only red any test can
produce, so refusing it would make the gate impassable for correct code, and a gate that
must always be waived teaches waiving.

⚠ **What it does not see**, said rather than implied: it judges **HEAD**, so a push of
another checkout's branch is judged by the wrong commit; it does not see `ghmerge`, which
merges by number with no tree here to hash; a record is keyed by the guarded file only, so
deleting the *test* that held a guard does not invalidate it; and it reads the command's
text, not a shell, so a push inside a quoted string or an `eval` is not seen.

**When it cannot read** — an unparseable payload, a repository with no `origin/HEAD` — it
**allows, and says so** to the user through the hook's `systemMessage`: an allowance that
is not announced would be the gate-that-does-nothing again. The waiver is
`MUTSWEEP_SKIP=1 gitpush …`, printed together with the files that went out unswept,
because a waiver is made in public or not at all.

### Nothing is written by truncation {#atomic}

⛔ **A restore on a signal is not enough, and it took a lost file to see it.** Measured
2026-11-21: a `mutsweep` was killed (exit 137 — **SIGKILL**) while sweeping
`go-fleettools/prwait`, and left `quota.go` at **zero bytes**. Not mutated, not
half-written: 434 lines gone, and `wc -l` answering 0. The cause was `os.WriteFile`,
which **truncates and then writes** — between those two steps the file is empty on disk,
and SIGKILL cannot be trapped, so the signal handler that restores on SIGINT could never
have covered it. The file happened to be committed, so nothing was lost. An hour of
uncommitted work would have been.

Since **v0.13.0** every mutation and every restore writes a temporary file **in the
target's own directory** and renames it over the target, so at each instant the file holds
either all of its old contents or all of the new ones, and a kill at any point leaves one
or the other. The temporary has to share that directory: a rename across filesystems is a
copy and an unlink, which would reintroduce exactly this window while looking like it had
closed it. The pattern was already in the repository, in `mutsweep/record.go` — the ledger
was written atomically and the source code was not.

⛔ **A swept file keeps the mode it had**, and that now takes doing. A rename installs a
*new inode*, which carries the temporary file's `0o600`, so an executable or a
group-readable source would have come back private after every mutation. The mode is read
off the target and handed back — read **once**, in the writer, rather than by each caller:
the callers that used to stat the file themselves each gained an error path that cannot be
reached, since a stat failing after the same file has just been read is a race no test can
arrange. `mutsweep` reported two of them as surviving, and it was right to.

The README also describes `-tally`, for planning and auditing a campaign swept in chunks,
and `-mutate boundary`, which replaces a range comparison instead of deleting it; read
them there.
