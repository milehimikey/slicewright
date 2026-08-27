---
marp: true
theme: default
paginate: true
size: 16:9
footer: 'Slicewright · em 1.8.0 / em-sdd-bridge 0.4.0'
---

<!-- _paginate: false -->
<!-- _footer: '' -->

# Slicewright

## agents propose, humans ratify, machines check

The Meridian Goods walkthrough — ~25 minutes, every command real

<!--
Cold open framing. Say both pinned version numbers out loud here —
em 1.8.0, em-sdd-bridge 0.4.0 — and leave them on screen (the footer)
for the rest of the talk.
-->

---

## What you're about to watch

- **Act I** — the model IS the spec (discover → model → gate → build)
- **Act II** — a change is a slice (add, change, or remove — nothing else)
- **Act III** — two loops that keep it honest (ratify, conform, ratify again)

Repository: a clean clone of `meridian-goods`, checkpointed as
`walkthrough/*` tags on real commits — nothing here is a screenshot.

<!--
LIVE/PRE-STAGED legend for the room: LIVE = a real command run on stage
in real time. PRE-STAGED = repo state prepared in advance so the
command still runs for real and the output is real — nothing here is a
screenshot or a mock.
-->

---

<!-- _class: lead -->
<!-- _paginate: false -->

# Act I
## The model is the spec

---

## Beat 0 — Cold open

**narration**

> "Meridian Goods needs software. Watch it get built — every command
> in this talk is real."

<!-- Proves: frames the promise the close (Beat 17) collects on. -->

<!--
narration, 0.5 min. One slide, no invocation. Frames the promise the
close collects on.
-->

---

## Beat 1 — Discover

**LIVE · ±rehearsal**

```
/event-modeling discover
```

A Socratic conversation, run against a fresh scratch file (never a
reuse of the finished model), live-produces the happy-path spine:
personas, a persona-facing command, the event it records.

<!-- Proves: this is a facilitated conversation with a tool, not code-first authorship — the agent never invents a domain fact the presenter didn't supply. -->

<!--
LIVE, 2.5 min ±rehearsal. Keep this small — one or two slices' worth
of events/commands. The point is proving the mechanism, not racing the
real model's size.
-->

---

## Beat 2 — Model

**LIVE · ±rehearsal**

```
/event-modeling model
```

The live draft gets grouped into contexts and classified into one of
the four patterns (State Change, State View, Automation, Translation)
— completeness-checked as it goes: every command needs a same-slice
trigger, every event needs a reader, every view needs a consumer.

> "A session like this one, run to completion, produced the model
> behind Meridian Goods — eight slices, all four patterns, no
> exceptions."

<!-- Proves: a "half-slice" is structurally rejected before it's ever allowed to exist — patterns aren't a naming convention, they're a gate. -->

<!--
LIVE, 2 min ±rehearsal. Close by pointing at the real repo, then
switch terminals/checkouts here.
-->

---

## Beat 3 — Live render

**LIVE**

```
git checkout walkthrough/01-model-ratified
em watch meridian-goods.em -o meridian-goods.svg --serve --port 4173
```

```
rendered meridian-goods.svg (99ms)
→ live view: http://localhost:4173/?svg=meridian-goods.svg
watching meridian-goods.em … (ctrl-c to stop)
```

This tag is real history: the commit where all eight base+change-beat
slice docs were ratified, before a line of Kotlin existed
(`git ls-tree` confirms zero `.kt` files here). At this tag only the
**base six** are ratified — the change-beat slices arrive at Beat 10.

<!-- Proves: the model is a live artifact a whole room can watch, not a private file — and "ratified" is a real git commit, not a slide's word. -->

<!--
LIVE, ~0.2 min render + narration. Open the served URL: one
self-contained page, inline-SVG pan/zoom, a Review mode toggle, live
reload (SSE). live.html/file:// viewing is gone as of 1.8.0. Don't
over-claim "all eight" at this tag — say "the base six."
-->

---

## Beat 4 — Slice deep-dive

**LIVE (place-order) / PRE-STAGED (the rest)**

Still on `walkthrough/01-model-ratified`. Open `slices/place-order.md`
— already fully written at this commit:

- Frontmatter: `schemaVersion`, `pattern`, `swimlane`, `status`, `version`
- Field table with type and rule per field
- Four named invariants: `INV-PO-1` … `INV-PO-4`
- Given/When/Then scenarios, one per invariant
- Open Questions — both already resolved and checked

<!-- Proves: the slice doc IS the spec — contract, business rules, and acceptance tests, captured once, in one file a human actually reads. -->

<!--
LIVE for place-order; the other seven slice docs are shown already
written, not authored on stage, so PRE-STAGED for those. 2.5 min.
-->

---

## Beat 5 — Validate, then break it on purpose

**LIVE**

```
em validate meridian-goods.em
→ ok — no issues
```

Break it: add `shippingSpeed: String` to the `Payments To Request`
view with no matching source event field, then:

```json
{
  "ok": true,
  "summary": { "errors": 0, "warnings": 1, "total": 1 },
  "diagnostics": [{
    "severity": "warning",
    "code": "fields-completeness/view-field-no-source",
    "message": "view \"Payments To Request\" field \"shippingSpeed\" has no source in \"Order Placed\"",
    "usageCategory": "view field no source"
  }]
}
```

<!-- Proves: a deterministic structural gate, not an LLM's judgment call — the same diagnostic, same code, same line, every time. -->

<!--
LIVE, ~0.5s per run. Note out loud: "ok": true even with a warning
present — errors and warnings are counted separately, export cares
only about errors. The --json flash is the point: a stable code and
usageCategory an agent can branch on, not a string it has to regex.
Restore the field before moving on (git checkout -- meridian-goods.em).
-->

---

## Beat 6 — Readiness gate: fail

**LIVE**

Locally (uncommitted): set `place-order.md`'s `status: draft`, uncheck
one Open Question.

```
em validate meridian-goods.em --slice-ready place-order --json
```

```json
{
  "sliceKey": "place-order",
  "gates": {
    "docBound": true, "frontmatterUsable": true,
    "statusReady": false, "noUncheckedOpenQuestions": false
  },
  "ready": false,
  "diagnostics": [
    { "code": "slice-ready-status-not-ready" },
    { "code": "slice-ready-open-questions-unchecked" }
  ]
}
```

<!--
LIVE, 2 min total for this beat's fail+pass pair. Still on
walkthrough/01-model-ratified.
-->

---

## Beat 6 — Readiness gate: pass

**LIVE**

A human checks the box back, flips `status: ready-to-implement` —
exactly what this file's real, unedited content already says:

```json
{
  "sliceKey": "place-order",
  "gates": {
    "docBound": true, "frontmatterUsable": true,
    "statusReady": true, "noUncheckedOpenQuestions": true
  },
  "ready": true,
  "diagnostics": []
}
```

All four gates named on screen: doc exists and is bound
(`docBound`), frontmatter parses (`frontmatterUsable`), status says
ready (`statusReady`), no open question unchecked
(`noUncheckedOpenQuestions`).

### "Agents propose, humans ratify" is mechanically enforced, not a slide.

<!-- Proves: the single cleanest demo beat in the toolchain. -->

<!--
LIVE. Restoring the file is restoring real history, not fabricating a
pass. Land the boxed line exactly.
-->

---

## Beat 7 — Bridge to spec-kit

**LIVE**

Switch to the demo clone at real `main` HEAD (code must already exist
— the bridge's precondition needs the event's Kotlin class already in
the source tree). Locally, revert `place-order.md`'s `status` from
`implemented` back to `ready-to-implement`.

```
npx -y em-sdd-bridge@0.4.0 place-order --symlink
```

```
Linked specs/001-place-order/spec.md -> ../../slices/place-order.md
Traceability: slice key(s) `place-order` · pattern `state-change`
```

```
ls -la specs/001-place-order/
lrwxr-xr-x  spec.md -> ../../slices/place-order.md
```

`FR-004 … INV-PO-2. The server recomputes and verifies this…`

<!-- Proves: "a spec IS a slice" — nothing generated, nothing duplicated; the ratified doc travels through spec-kit's own phases as-is. -->

<!--
LIVE, ~2.5s (npx cold). file specs/001-place-order/spec.md confirms a
symlink, not a generated file. Every functional requirement carries a
literal INV-<MNEMONIC>-<n> back-reference, so spec-kit's own
plan/tasks/implement phases run against the ratified slice doc itself.
-->

---

## Beat 7b — Governance by construction *(optional)*

**LIVE**

On the same clone, set `cancel-order.md`'s `status` to `draft`. Try to
bridge it:

```
npx -y em-sdd-bridge@0.4.0 cancel-order --symlink
```

```
warn :95 slice "cancel-order" is status: draft, not ready-to-implement
bridge: `em validate … --slice-ready cancel-order` reports "cancel-order"
is not ready to implement
slice "cancel-order" is NOT ready-to-implement
```

<!-- Proves: governance enforced by the model's own structure, not by review discipline — one refusal makes the enterprise-legitimacy case. -->

<!--
LIVE, ~1 min. The bridge delegates to em's own readiness gate as its
precondition rather than reimplementing the check — it can't be fooled
into bridging a slice a human hasn't ratified. Restore cancel-order.md
(git checkout -- slices/cancel-order.md) before continuing. Optional:
cut first if running long (Beat 6 already proves the ratification-gate
point).
-->

---

## Beat 8 — Agent implements, live mark-implemented

**LIVE**

A PR is open implementing `place-order`: the slice doc read as
read-only spec except two fields, gaps surfaced to a human, tests
citing `INV-PO-*`.

```
em slice mark-implemented meridian-goods.em place-order \
  https://github.com/milehimikey/meridian-goods/pull/7
→ marked implemented: slices/place-order.md (implementedIn: .../pull/7)
```

Idempotent rerun: `→ already implemented (no-op)`

Conflict refusal (different PR URL):
`→ refusing to overwrite`

<!-- Proves: a 20-second provenance-governance moment — implementation status is flipped by one auditable command, and it cannot be silently corrupted by a second, conflicting claim. -->

<!--
LIVE, ~20s. That first run used the real PR #7 URL, so the file the
demo just edited is now byte-identical to real, unedited main HEAD —
the revert at Beat 7 and the mark-implemented here cancel out exactly.
-->

---

## Beat 9 — Export peek

**LIVE**

Back on real `main` HEAD (state fully converged after Beat 8):

```
em export meridian-goods.em --slice place-order
```

```json
{
  "schemaVersion": "1.6",
  "sliceKey": "place-order",
  "slice": {
    "pattern": "state-change",
    "doc": {
      "status": "implemented", "version": 1,
      "implementedIn": "https://github.com/milehimikey/meridian-goods/pull/7",
      "driftSignal": "in-sync"
    }
  }
}
```

<!-- Proves: the slice lifecycle is machine-readable, not folklore in a markdown file — sets up the conform beat. -->

<!--
LIVE, ~0.25s. Scoped to one slice, not the whole model — 169 lines
instead of the full export. This is the same normalized document an
implementing agent reads to work, and the same one conform compares
against the codebase later.
-->

---

## Beat 9b — Same JSON, two transports *(optional)*

**LIVE**

`em`'s MCP server is the same npm package's second bin:

```
em-mcp
```

`tools/list` returns exactly 7 read-only tools: `validate`,
`slice_ready`, `list_markers`, `export_model`, `export_slice`,
`coverage`, `contract`. Call `slice_ready` over MCP and diff it
against the CLI's `--json` output, same model, same moment:
**byte-identical** (modulo a trailing newline).

<!-- Proves: "one schema per surface" — an MCP client and a shell script parsing --json see the same document, because both call the same JSON-builder code underneath. -->

<!--
LIVE, ~1 min. Optional — cut second if running long (a strong but
non-load-bearing claim; Beat 7's symlink already proves the sharper
one).
-->

---

<!-- _class: lead -->
<!-- _paginate: false -->

# Act II
## Change is add, change, or remove — nothing else

---

## Beat 10 — Change arrives

**LIVE · ±rehearsal**

```
git checkout walkthrough/03-all-base-implemented
```

"Customers have been phoning support to cancel orders they placed by
mistake." Add `Cancel Order` (State Change) plus a **second,
independent** instance of `Open Orders` fed by the new event.

```
/event-modeling slice
```

Draw it live: the two `Open Orders` instances share a name and take
**no arrow** between them — repeated views never connect to each
other, each folds from its own events.

<!-- Proves: a change request is modeled as add a slice, not a code-first patch — and the repeated-view rule is enforced live, on-model, not asserted in a slide. -->

<!--
LIVE, 2.5 min ±rehearsal. This session's real end state is
checkpointed at walkthrough/04-change-beat.
-->

---

## Beat 11 — Structural diff

**LIVE**

```
em diff meridian-goods.em --from 238d324 --to 7230315
```

```
2 slices added

+ slice "Cancel Order"
+ slice "Open Orders — cancelled"
```

These are the two real commits either side of the real change-beat PR
(#13 → #14) — not a synthetic pair. `--json` carries
`counts.slicesAdded: 2`, each change typed `slice-added` with its
`sliceKey`.

<!-- Proves: the diff speaks in model vocabulary — slices, not YAML noise — because every element ref survived the edit unchanged. -->

<!--
LIVE, ~0.4s.
-->

---

## Beat 12 — Business changelog

**LIVE**

```
em changelog meridian-goods.em
```

```
## 2026-08-23 — Change beat: Cancel Order + repeated Open Orders
view (model + docs) (#14) (7230315)

2 slices added

+ slice "Cancel Order"
+ slice "Open Orders — cancelled"
```

<!-- Proves: the audit trail an enterprise buyer can actually read — the governance payoff made concrete. -->

<!--
LIVE, ~0.3s. Full git-history walk, rendered newest-first as a
business-readable ledger — a non-engineer stakeholder can read this
and know what changed and why, without opening a diagram or a diff.
-->

---

<!-- _class: lead -->
<!-- _paginate: false -->

# Act III
## Two loops that keep it honest

---

## Beat 13 — Time skip

**narration**

> "Three weeks later, an engineer shipped a real feature under time
> pressure: customers were asking why a cancel right after checkout
> still got rejected once payment cleared, so a 24-hour grace window
> went in — without anyone touching the model or the doc first."

<!-- Proves: sets up the drift story as ordinary, not villainous — this is what drift actually looks like: a reasonable change, shipped fast, doc untouched. -->

<!--
narration, 0.5 min.
-->

---

## Beat 14 — Conform: evidence-cited report

**PRE-STAGED (real report, real commits)**

```
git checkout walkthrough/06-conformance-report
```

> `CancelOrderCommandHandler.kt:44-47` — `Duration.ofHours(24)`. The
> INV-CO-1 check now allows cancellation up to 24 hours after
> `Payment Captured`. Both `slices/cancel-order.md`'s INV-CO-1 text and
> `domain-decisions.md`'s cancellation bullet still say capture is an
> absolute, exception-free bar.

`em diff --json`: `"identical": true` — **zero structural drift**.
This drift is entirely behavioral, invisible to a structural diff
alone.

<!-- Proves: drift is caught with cited evidence — file, line, a literal code snippet — not asserted, and the maturity of that evidence is stated at its own size, not oversold. -->

<!--
PRE-STAGED, 2.5 min. This is real: PR #17 shipped the grace window
before the model or doc changed. The first conform run on this
repository found it. State the honesty caveat exactly, out loud, every
time this beat runs (see the next slide's exact wording) — do not
paraphrase looser than that on stage.
-->

---

## Honest about the evidence: two runs, dated

| Date | Slices | Invariants | Findings | False positives |
|---|---|---|---|---|
| 2026-07 | 19 | 58 | 10, incl. real drift | 0 |
| 2026-08 | 8 | 20* | 4 — 1 seeded, 3 organic | 0 |

Both runs caught real drift nobody planted and produced zero false
positives. Both were ratified same-cycle, with dated rulings.

> *"That's real, and worth saying plainly — but it's two runs, not a
> pattern, and the honest version of this claim keeps saying 'two
> runs, dated' for as long as that's true."*

<!--
Say this caveat exactly, every time this beat runs — do not paraphrase
looser than the exact wording above. This is now two dated data
points, not a track record. A methodology that ships a drift-checker
had better not drift from its own claims.
-->

---

## Beat 15 — Ratify: issue → divergence

**PRE-STAGED, disclosed as a staged illustration**

*"Real ratification here happened as prose — a report, a dated
ruling, a doc edit — not as an in-model marker; nothing in this
repository's actual history ever used em's issue/divergence DSL
markers. What you're about to see is a staged illustration of the
mechanism those markers give you, using the exact same finding."*

```
em validate meridian-goods.em
→ warn: open issue on "Order Cancelled": conformance: code allows
  cancel within a 24h grace window after Payment Captured …

# ratified →
em validate meridian-goods.em
→ ok — no issues
```

<!-- Proves: ratified decisions stop being re-litigated — the exact "alert fatigue" pain governance buyers care about, solved with two marker states, illustrated honestly as staged, backed by a real ratification record right next to it. -->

<!--
PRE-STAGED, disclosed as staged, 1.5 min. STATE THE DISCLOSURE
DISCLAIMER ABOVE, VERBATIM, EVERY TIME THIS BEAT RUNS — this is the
one point in the talk where the visual is a staged illustration rather
than a screen capture of real history, and the honesty bar requires
saying so out loud on stage, not just in a footnote. Then show what
real ratification actually produced: slices/cancel-order.md jumped
version: 1 -> 2, INV-CO-1 re-typed in place, a Delta block records the
origin (PR #17), the detection, and the ruling.
-->

---

## Beat 16 — Governance surface *(optional)*

**LIVE**

```
em validate meridian-goods.em --list-divergences --json
```
(real `main` HEAD returns empty `markers: []` — say this plainly,
real ratification here never used the DSL marker)

```
em coverage meridian-goods.em --tests src/test --strict
→ 25 invariant(s) checked, 0 uncovered
```

Every `INV-*` id across all 8 implemented slices cited with
file:line. `AGENTS.md`'s managed section points any agent at
`em contract`, the 4-gate check, and the MCP alternative.

<!-- Proves: the whole model's decision/audit surface is queryable from a script, not just eyeballed on one diagram. -->

<!--
LIVE, ~1 min. Optional — cut first if running long (restated by Beat
12's changelog and Beat 9's export peek anyway).
-->

---

<!-- _class: lead -->
<!-- _paginate: false -->

# Four pillars
## What an architecture review board actually asks

---

## Pillar 1 — Human-ratification gates

**The fear**: an agent ships scope nobody signed off on.

**The mechanism**: `em validate --slice-ready` is not a convention an
agent could route around — `em-sdd-bridge` delegates to this exact
command as its own precondition, and the implementing agent's own
contract requires gating on it before touching code. On Meridian
Goods, this gate ran on all eight slices before a line of Kotlin
existed.

> "Agents propose. Humans ratify."

<!-- Docs maxim, cited from process docs. -->

---

## Pillar 2 — Evidence-cited drift reports, never auto-applied

**The fear**: an AI silently rewrites your system of record based on
its own inference.

**The mechanism**: `em conform` produces a structured report — file
and line citations, a classification per finding — that proposes,
never applies. Nothing in the loop writes to the ratified model
without a human ratifying the finding.

---

## Pillar 3 — Decision lifecycle and lineage

**The fear**: how do we know a decision was actually decided, by
whom, and why — and how do we stop re-litigating it every audit
cycle?

**The mechanism**: `issue "..."` renders red on the diagram; ratified,
it re-types to `divergence "..."` (teal), which never warns again.
Every slice doc carries `status`, `version`, and lineage keys. `em
changelog` renders the same history as a business-readable ledger.

---

## Pillar 4 — Deterministic core, agentic edges

**The fear**: is the tool itself trustworthy, or another
probabilistic black box we now have to audit?

**The mechanism**: no LLM call anywhere inside `em`'s CLI;
byte-deterministic output for identical input — confirmed both
between two runs of the same command, and between the CLI and the MCP
server reading the same model at the same moment. Plain text,
git-native, works offline.

---

## Beat 17 — Close

**narration**

> "Every command you just watched was real, against pinned versions,
> on a repository anyone can clone. Nothing in this talk was a slide
> pretending to be software."

<!-- Proves: names the demonstrated-mechanics bar explicitly, closing the loop Beat 0 opened. -->

<!--
narration, 0.5 min.
-->

---

<!-- _paginate: false -->

# Try it

MIT-licensed. One npm package. No account, no hosted service.

```
git clone <meridian-goods repo>
npm install -g @milehimikey/em
em init
```

**This is a pilot invitation, not a product launch.** If your team
already runs an SDD tool, the ask is small: point the bridge at one
slice, watch it refuse an unratified one, watch `--symlink` hand a
real spec-kit phase a document it never had to write. If you don't,
the whole loop — model, gate, implement, conform, ratify — is the
complete pipeline.

<!--
Close slide. Repo pointer + the pilot invitation line, drawn from the
"early, credible, tryable" narrative beat. Don't oversell — this is an
invitation to try it, not a sales pitch.
-->
