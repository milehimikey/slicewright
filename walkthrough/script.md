# The Meridian Goods walkthrough — locked script

**Runtime**: ~25 minutes core (Beats 0–17), ~28 minutes with both optional
beats (7b, 9b, 16). **Pinned versions**: `em 1.8.0`, `em-sdd-bridge 0.4.0` —
say both version numbers out loud at Beat 0 or Beat 3 and leave them on
screen (a terminal prompt or a slide corner) for the rest of the talk.

**Repository**: a clean clone of `meridian-goods`. Set repo-local git
identity in every fresh clone before any commit
(`git config user.name` / `user.email`) — this repo is the proof artifact;
never demo from a clone carrying the wrong author. See
`presenter-checklist.md` for the full pre-show setup, including the two
places this script deliberately steps off real `main` HEAD and why.

**Legend**: **LIVE** = a real command, run on stage, in real time.
**PRE-STAGED** = repo state prepared in advance (a checkout, a scratch
edit, a seeded marker) so the *command* still runs for real and the
*output* is real — nothing here is a screenshot or a mock. **±rehearsal**
next to a time means: this beat's duration is a live conversation with the
audience, not a scripted invocation: time it in rehearsal, don't trust the
estimate on stage.

Every "Proves" line is what the audience should walk away believing. Every
output block below was captured from a real run against the pinned
versions; it is trimmed for stage, never invented.

---

## Beat 0 — Cold open

**0.5 min · narration**

One slide: "Meridian Goods needs software. Watch it get built — every
command in this talk is real."

**Proves**: frames the promise the close (Beat 17) collects on.

---

## Beat 1 — Discover

**2.5 min ±rehearsal · LIVE**

A Socratic conversation, run against a fresh scratch file (not the
`meridian-goods` repo — the demo model is authored live, never a reuse of
the finished one), live-produces the happy-path spine: personas, a
persona-facing command, the event it records. Keep this small — one or two
slices' worth of events/commands — since the point is proving the
mechanism, not racing the real model's size.

```
/event-modeling discover
```

**Proves**: this is a facilitated conversation with a tool, not code-first
authorship — the agent never invents a domain fact the presenter didn't
supply.

---

## Beat 2 — Model

**2 min ±rehearsal · LIVE**

The live draft gets grouped into contexts and classified into one of the
four patterns (State Change, State View, Automation, Translation) —
completeness-checked as it goes: every command needs a same-slice trigger,
every event needs a reader, every view needs a consumer.

```
/event-modeling model
```

Close the beat by pointing at the real repo: "a session like this one,
run to completion, produced the model behind Meridian Goods — eight
slices, all four patterns, no exceptions. That's what the rest of this
talk stands on." Switch terminals/checkouts here.

**Proves**: a "half-slice" is structurally rejected before it's ever
allowed to exist — patterns aren't a naming convention, they're gate.

---

## Beat 3 — Live render

**~0.2 min render + narration · LIVE**

```
git checkout walkthrough/01-model-ratified
em watch meridian-goods.em -o meridian-goods.svg --serve --port 4173
```

```
rendered meridian-goods.svg (99ms)
→ live view: http://localhost:4173/?svg=meridian-goods.svg
  open it in a browser and share your screen
watching meridian-goods.em … (ctrl-c to stop)
```

This tag is real history: the commit where all eight base+change-beat
slice docs* were ratified, before a line of Kotlin existed (`git ls-tree`
confirms zero `.kt` files at this commit — say so, it's a nice beat).
Open the served URL: one self-contained page, inline-SVG pan/zoom, a
"Review" mode toggle, and a live-reload connection (`EventSource`/SSE) —
`live.html`/file:// viewing is gone as of 1.8.0; there's exactly one served
URL now.

*(the change-beat slices — Cancel Order, the repeated Open Orders — arrive
later at Beat 10; at this tag only the six base slices are ratified. Don't
over-claim "all eight" here; say "the base six," or don't count out loud.)*

**Proves**: the model is a live artifact a whole room can watch, not a
private file — and "ratified" is a real git commit, not a slide's word.

---

## Beat 4 — Slice deep-dive

**2.5 min · LIVE (place-order) / PRE-STAGED (the rest)**

Stay on `walkthrough/01-model-ratified`. Open `slices/place-order.md` —
already fully written at this commit. Walk the frontmatter (5 required
keys: `schemaVersion`, `pattern`, `swimlane`, `status`, `version`), the
field table, the four named invariants (`INV-PO-1`…`INV-PO-4`), the GWT
scenarios, the Open Questions section (both already resolved and checked
here — that matters at Beat 6). The other seven slice docs are shown
already written, not authored on stage.

**Proves**: the slice doc *is* the spec — contract, business rules, and
acceptance tests, captured once, in one file a human actually reads.

---

## Beat 5 — Validate, then break it on purpose

**~0.5s per run · LIVE**

```
em validate meridian-goods.em
→ ok — no issues
```

Now break it — **add** a view field with no matching source event field
(not delete one: deleting a mirrored field is a silent no-op in em 1.8.0,
so the break has to go the other direction). Add `shippingSpeed: String`
to the `Payments To Request` view:

```
em validate meridian-goods.em --json
```
```json
{
  "validateSchemaVersion": "1.0",
  "generator": { "name": "@milehimikey/em", "version": "1.8.0" },
  "file": "meridian-goods.em",
  "ok": true,
  "summary": { "errors": 0, "warnings": 1, "total": 1 },
  "diagnostics": [
    {
      "severity": "warning",
      "code": "fields-completeness/view-field-no-source",
      "message": "view \"Payments To Request\" field \"shippingSpeed\" has no source in \"Order Placed\"",
      "line": 33,
      "refs": ["payments-to-request/view.payments-to-request", "place-order/event.order-placed"],
      "usageCategory": "view field no source"
    }
  ]
}
```

Note out loud: `"ok": true` even with a warning present — errors and
warnings are counted separately, and export cares only about errors (see
the asymmetry below). The `--json` flash is the beat's real point: a
stable `code` and `usageCategory` an agent can branch on, not a string an
agent has to regex. Restore the field before moving on
(`git checkout -- meridian-goods.em`).

**Proves**: a deterministic structural gate, not an LLM's judgment call —
the same diagnostic, same code, same line, every time.

---

## Beat 6 — Readiness gate: fail → pass

**2 min · LIVE**

Still on `walkthrough/01-model-ratified`. Locally (uncommitted) set
`place-order.md`'s frontmatter to `status: draft` and uncheck one of its
two Open Questions, to show a real failure:

```
em validate meridian-goods.em --slice-ready place-order --json
```
```json
{
  "validateSliceReadySchemaVersion": "1.0",
  "generator": { "name": "@milehimikey/em", "version": "1.8.0" },
  "file": "meridian-goods.em",
  "sliceKey": "place-order",
  "gates": {
    "docBound": true,
    "frontmatterUsable": true,
    "statusReady": false,
    "noUncheckedOpenQuestions": false
  },
  "ready": false,
  "diagnostics": [
    { "severity": "warning", "code": "slice-ready-status-not-ready",
      "message": "slice \"place-order\" is status: draft, not ready-to-implement",
      "line": 14, "refs": ["place-order"],
      "usageCategory": "slice-ready slice not ready-to-implement" },
    { "severity": "warning", "code": "slice-ready-open-questions-unchecked",
      "message": "slice \"place-order\" has 1 of 2 Open Question(s) unchecked",
      "line": 14, "refs": ["place-order"],
      "usageCategory": "slice-ready slice has unchecked open questions" }
  ]
}
```

A human checks the box back and flips `status: ready-to-implement` — which
is exactly what this file's real, unedited content *at this commit*
already says, so restoring it is restoring real history, not fabricating a
pass:

```
em validate meridian-goods.em --slice-ready place-order --json
```
```json
{
  "validateSliceReadySchemaVersion": "1.0",
  "generator": { "name": "@milehimikey/em", "version": "1.8.0" },
  "file": "meridian-goods.em",
  "sliceKey": "place-order",
  "gates": { "docBound": true, "frontmatterUsable": true, "statusReady": true, "noUncheckedOpenQuestions": true },
  "ready": true,
  "diagnostics": []
}
```

Name all four gates on screen: doc exists and is bound to the slice
(`docBound`), frontmatter parses (`frontmatterUsable`), status says ready
(`statusReady`), no open question is still unchecked
(`noUncheckedOpenQuestions`). All four, not one flag.

**Proves**: **"agents propose, humans ratify" is mechanically enforced,
not a slide.** The single cleanest demo beat in the toolchain.

---

## Beat 7 — Bridge to spec-kit

**~2.5s (npx cold) · LIVE**

Switch to the demo clone at current `main` (real code exists there — the
bridge's own precondition needs the event's Kotlin class to already be in
the source tree, which only a merged repo has; say this plainly, it's an
honest mechanical fact about the tool, not a demo shortcut). On that
clone, locally (uncommitted) revert `place-order.md`'s `status` from
`implemented` back to `ready-to-implement` — the exact value the doc held
before PR #7 merged it.

```
npx -y em-sdd-bridge@0.4.0 place-order --symlink
```
```
Linked specs/001-place-order/spec.md -> ../../slices/place-order.md on branch 001-place-order
Traceability: slice key(s) `place-order` · pattern `state-change` · model `meridian-goods.em` · slice doc(s): `slices/place-order.md`
```

```
ls -la specs/001-place-order/
lrwxr-xr-x  spec.md -> ../../slices/place-order.md
```

`file specs/001-place-order/spec.md` confirms a symlink, not a generated
file. Open it: every functional requirement carries a literal
`INV-<MNEMONIC>-<n>` back-reference —

```
FR-001 … see INV-PO-1 and domain-decisions.md#client-generated-orderid-for-idempotency
FR-004 … INV-PO-2. The server recomputes and verifies this…
```

— so spec-kit's own `plan`/`tasks`/`implement` phases run against the
ratified slice doc itself, never a paraphrase of it.

**Proves**: **"a spec IS a slice"** — nothing generated, nothing
duplicated; the ratified doc travels through spec-kit's own phases as-is.

---

## Beat 7b — Governance by construction *(optional, ~1 min)*

**LIVE**

On the same clone, locally set `cancel-order.md`'s `status` to `draft`.
Try to bridge it:

```
npx -y em-sdd-bridge@0.4.0 cancel-order --symlink
```
```
warn :95 slice "cancel-order" is status: draft, not ready-to-implement
bridge: `em validate … --slice-ready cancel-order` reports "cancel-order" is not ready to implement:
warn :95 slice "cancel-order" is status: draft, not ready-to-implement
slice "cancel-order" is NOT ready-to-implement
```

The bridge delegates to `em`'s own readiness gate as its precondition
rather than reimplementing the check — it can't be fooled into bridging a
slice a human hasn't ratified. Restore `cancel-order.md`
(`git checkout -- slices/cancel-order.md`) before continuing.

**Proves**: governance enforced by the model's own structure, not by
review discipline — one refusal makes the enterprise-legitimacy case.

---

## Beat 8 — Agent implements, LIVE mark-implemented

**~20s · LIVE**

Narrate: a PR is open (screen it, real or pre-staged) implementing
`place-order` under `reference/implement.md`'s contract — the doc read as
read-only spec except two fields, gaps surfaced to a human instead of
silently decided, tests citing `INV-PO-*`. At merge, the only thing that
changes is those two frontmatter fields — and that flip now runs live, on
the exact clone Beat 7 revert left in `ready-to-implement`:

```
em slice mark-implemented meridian-goods.em place-order https://github.com/milehimikey/meridian-goods/pull/7
```
```
marked implemented: slices/place-order.md (implementedIn: https://github.com/milehimikey/meridian-goods/pull/7)
```

Idempotent rerun — merging twice, or a CI retry, is a no-op:

```
em slice mark-implemented meridian-goods.em place-order https://github.com/milehimikey/meridian-goods/pull/7
→ already implemented (no-op): slices/place-order.md
```

Conflict refusal — a *different* PR URL for an already-marked slice is
provenance corruption, and the tool refuses it outright:

```
em slice mark-implemented meridian-goods.em place-order https://github.com/milehimikey/meridian-goods/pull/999
→ em slice mark-implemented: slices/place-order.md: already marked implemented
  with a different URL (existing: .../pull/7, requested: .../pull/999) —
  refusing to overwrite
```

Note for the room: that first run used the real PR #7 URL, so the file the
demo just edited is now byte-identical to real, unedited `main` HEAD — the
revert at Beat 7 and the mark-implemented here cancel out exactly.

**Proves**: a 20-second provenance-governance moment — implementation
status is flipped by one auditable command, and it cannot be silently
corrupted by a second, conflicting claim.

---

## Beat 9 — Export peek

**~0.25s · LIVE**

Back on real `main` HEAD (state fully converged after Beat 8):

```
em export meridian-goods.em --slice place-order
```
```json
{
  "schemaVersion": "1.6",
  "generator": { "name": "@milehimikey/em", "version": "1.8.0" },
  "sliceKey": "place-order",
  "slice": {
    "key": "place-order", "name": "Place Order", "pattern": "state-change",
    "doc": {
      "found": true, "path": "slices/place-order.md",
      "status": "implemented", "version": 1,
      "implementedIn": "https://github.com/milehimikey/meridian-goods/pull/7",
      "driftSignal": "in-sync"
    },
    "elements": [ "…" ]
  }
}
```

Scoped to one slice, not the whole model — `169` lines instead of the full
export. This is the same normalized document an implementing agent reads
to work, and the same one `conform` compares against the codebase later.

**Proves**: the slice lifecycle is machine-readable, not folklore in a
markdown file — sets up the conform beat.

---

## Beat 9b — Same JSON, two transports *(optional, ~1 min)*

**LIVE**

`em`'s MCP server is the same npm package's second bin, not a separate
install:

```
em-mcp
```

Over stdio: `initialize` names `serverInfo: {"name":"em","version":"1.8.0"}`.
`tools/list` returns exactly 7 read-only tools: `validate`, `slice_ready`,
`list_markers`, `export_model`, `export_slice`, `coverage`, `contract`.
Call `slice_ready` for `place-order` over MCP and diff its `content[0].text`
against the CLI's `--json` stdout from the same model, same moment:
**byte-identical**, modulo the CLI's single trailing newline (a framing
artifact of the two transports, not a content difference).

**Proves**: "one schema per surface" — an MCP client and a shell script
parsing `--json` see the same document, because both call the same
JSON-builder code underneath.

---

## Beat 10 — Change arrives

**2.5 min ±rehearsal · LIVE**

```
git checkout walkthrough/03-all-base-implemented
```

"Customers have been phoning support to cancel orders they placed by
mistake." Re-enter the `slice` phase live and add `Cancel Order` (State
Change: `ui Account Page @Customer`, `command Cancel Order`, `event Order
Cancelled @Ordering`) plus a **second, independent** instance of `Open
Orders` fed by the new event (`view Open Orders again from "Order
Cancelled"`, its own `ui`). Draw it on the live-rendered diagram and point
out explicitly: the two `Open Orders` instances share a name and take **no
arrow** between them — repeated views never connect to each other, each
one folds from its own events. This session's real end state is checkpointed
at `walkthrough/04-change-beat`.

```
/event-modeling slice
```

**Proves**: a change request is modeled as *add a slice*, not a
code-first patch — and the repeated-view rule is enforced live, on-model,
not asserted in a slide.

---

## Beat 11 — Structural diff

**~0.4s · LIVE**

```
em diff meridian-goods.em --from 238d324 --to 7230315
```
```
2 slices added

+ slice "Cancel Order"
+ slice "Open Orders — cancelled"
```

These are the two real commits either side of the real change-beat PR
(#13 → #14) — not a synthetic pair. `--json` carries the same facts
structurally (`counts.slicesAdded: 2`, each change typed `slice-added`
with its `sliceKey`) for a script or CI gate to consume.

**Proves**: the diff speaks in model vocabulary — slices, not YAML noise —
because every element ref survived the edit unchanged.

---

## Beat 12 — Business changelog

**~0.3s · LIVE**

```
em changelog meridian-goods.em
```
```
# Model changelog — meridian-goods.em

## 2026-08-23 — Change beat: Cancel Order + repeated Open Orders view (model + docs) (#14) (7230315)

2 slices added

+ slice "Cancel Order"
+ slice "Open Orders — cancelled"

## 2026-08-23 — Reconcile timestamp origins + ratify all six base slices (#4) (c858bbe)
…
```

Full git-history walk, rendered newest-first as a business-readable
ledger — a non-engineer stakeholder can read this and know what changed
and why, without opening a diagram or a diff.

**Proves**: the audit trail an enterprise buyer can actually read — the
governance payoff made concrete.

---

## Beat 13 — Time skip

**0.5 min · narration**

"Three weeks later, an engineer shipped a real feature under time
pressure: customers were asking why a cancel right after checkout still
got rejected once payment cleared, so a 24-hour grace window went in —
without anyone touching the model or the doc first."

**Proves**: sets up the drift story as ordinary, not villainous — this is
what drift actually looks like: a reasonable change, shipped fast, doc
untouched.

---

## Beat 14 — Conform: evidence-cited report

**2.5 min · PRE-STAGED (real report, real commits)**

```
git checkout walkthrough/06-conformance-report
```

Open `conformance/2026-08-23-report.md` — this is real: PR #17 shipped a
24-hour post-capture grace window to `Cancel Order` before the model or
doc changed. The first conform run on this repository found it:

> `CancelOrderCommandHandler.kt:44-47` — `val CANCELLATION_GRACE: Duration
> = Duration.ofHours(24)`. The INV-CO-1 check now allows cancellation up
> to 24 hours after `Payment Captured`. Both `slices/cancel-order.md`'s
> INV-CO-1 text and `domain-decisions.md`'s cancellation bullet still say
> capture is an absolute, exception-free bar.

Point out `em diff --json`'s companion fact on screen: `"identical":
true` — **zero structural drift**. Every field and flow still matches the
model exactly; this drift is entirely behavioral, invisible to a
structural diff alone, which is exactly why a conform run exists.

State the caveat exactly, out loud, every time this beat runs: *"This is
now two dated data points, not a track record — 2026-07's first validated
run (19 slices, 58 invariants, 10 findings, 0 false positives) and this
one (8 slices, 25 invariants, 4 findings — 1 shaped like that same real
finding, 3 nobody planted, 0 false positives). Both caught real drift and
both were ratified same-day. That's real, and worth saying plainly — but
it's two runs, not a pattern, and the honest version of this claim keeps
saying 'two runs, dated' for as long as that's true."* (exact wording from
`narrative/09-early-credible-tryable.md` — do not paraphrase looser than
this on stage.)

**Proves**: drift is caught with **cited evidence** — file, line, a
literal code snippet — not asserted, and the maturity of that evidence is
stated at its own size, not oversold.

---

## Beat 15 — Ratify: issue → divergence

**1.5 min · PRE-STAGED, disclosed as a staged illustration**

Honest framing for the room: *"Real ratification here happened as prose —
a report, a dated ruling, a doc edit — not as an in-model marker; nothing
in this repository's actual history ever used `em`'s `issue`/`divergence`
DSL markers. What you're about to see is a staged illustration of the
mechanism those markers give you, using the exact same finding."*

On a scratch copy of the model, seed the open question the way an agent
proposing it would:

```em
event Order Cancelled @Ordering issue "conformance: code allows cancel within
  a 24h grace window after Payment Captured; doc and domain-decisions.md both
  still say capture is an absolute bar" { … }
```
```
em validate meridian-goods.em
→ warn :102 open issue on "Order Cancelled": conformance: code allows
  cancel within a 24h grace window after Payment Captured; doc and
  domain-decisions.md both still say capture is an absolute bar
```

A human reads the evidence (the real Beat 14 report), agrees the grace
window was an intentional, undocumented call, and ratifies it — the
`issue` becomes a `divergence`:

```em
event Order Cancelled @Ordering divergence "grace window ratified
  2026-08-23: INV-CO-1 v2 allows cancel within 24h of Payment Captured;
  slices/cancel-order.md and domain-decisions.md updated to match" { … }
```
```
em validate meridian-goods.em
→ ok — no issues
```

Re-render: the marker's diagram corner flips from a red open-question fold
to a teal accepted-divergence fold, and it **never raises a validate
warning again** — that's the whole point of the second state. Then show
what real ratification actually produced in this repo, side by side:
`slices/cancel-order.md` jumped `version: 1 → 2`, INV-CO-1 was re-typed
in place, and a `## Delta` block records the origin (PR #17), the
detection (this conform run), and the ruling — a reader six months from
now doesn't need git archaeology to learn the rule changed, when, or why.

**Proves**: **ratified decisions stop being re-litigated** — the exact
"alert fatigue" pain governance buyers care about, solved with two marker
states, illustrated honestly as staged, backed by a real ratification
record right next to it.

---

## Beat 16 — Governance surface *(optional, ~1 min)*

**LIVE**

```
em validate meridian-goods.em --list-divergences --json
```
(on the seeded scratch copy from Beat 15 — real `main` HEAD returns an
empty `markers: []`, since real ratification here never used the DSL
marker; say this plainly rather than letting an empty result look broken)

```
em coverage meridian-goods.em --tests src/test --strict
→ 25 invariant(s) checked, 0 uncovered
```

Every `INV-*` id across all 8 implemented slices cited with file:line.
Glance at `AGENTS.md`'s managed section
(`<!-- GENERATED:agent-contract:start/end -->`) — auto-written by
`em skill install`, pointing any agent at `em contract`, the 4-gate check,
and the MCP alternative.

**Proves**: the whole model's decision/audit surface is queryable from a
script, not just eyeballed on one diagram.

---

## Beat 17 — Close

**0.5 min · narration**

"Every command you just watched was real, against pinned versions, on a
repository anyone can clone. Nothing in this talk was a slide pretending
to be software."

**Proves**: names the demonstrated-mechanics bar explicitly, closing the
loop Beat 0 opened.

---

## Cut order if running long

Drop **16** first (restated by Beat 12's changelog and Beat 9's export
peek anyway) → then **9b** (a strong but non-load-bearing claim; Beat 7's
symlink already proves the sharper one) → then **7b** (real, but Beat 6
already proves the ratification-gate point) → then trim Beat 4 to a
shorter field/invariant subset instead of the full doc. Core beats
(0–6, 7–9, 10–15, 17) do not compress further without cutting a proof
point.
