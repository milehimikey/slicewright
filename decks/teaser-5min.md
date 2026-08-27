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

<!--
Cold open. Say both version numbers out loud here and leave them on
screen (footer) for the rest of the talk: em 1.8.0, em-sdd-bridge 0.4.0.
-->

---

# The whole loop, in five minutes

**model → gate → build → conform → ratify**

This compresses the full walkthrough's two-loop arc — the demand loop
(model, gate) and the supply loop (build, conform, ratify) — into one
pass. The diagram already exists; no live Socratic authoring here.

<!--
This talk assumes the diagram already exists. Full detail, mode markers,
and exact invocations live in the full 25-minute walkthrough script;
this deck only tightens the timing.
-->

---

## 1. Flash the finished diagram

**PRE-STAGED**

```
git checkout walkthrough/01-model-ratified
em watch meridian-goods.em -o meridian-goods.svg --serve --port 4173
```

```
rendered meridian-goods.svg (99ms)
→ live view: http://localhost:4173/?svg=meridian-goods.svg
watching meridian-goods.em … (ctrl-c to stop)
```

> "Here's Meridian Goods' ordering process, modeled as a timeline —
> six ratified slices, four patterns, zero lines of code yet."

<!--
PRE-STAGED: repo state prepared in advance, but the command runs for
real and the output is real. Open the served URL, pan across it once.
Don't narrate the render mechanics.
-->

---

## 2. Gate: fail

**LIVE**

```
em validate meridian-goods.em --slice-ready place-order --json
```

```json
{
  "sliceKey": "place-order",
  "gates": {
    "statusReady": false,
    "noUncheckedOpenQuestions": false
  },
  "ready": false
}
```

<!--
LIVE. Same sequence as the full script's readiness-gate beat, trimmed
to the two --slice-ready calls. Skip naming all four gates individually
here — just show ready:false.
-->

---

## 2. Gate: pass

**LIVE** — a human checks the box, flips `status: ready-to-implement`

```
em validate meridian-goods.em --slice-ready place-order --json
```

```json
{
  "sliceKey": "place-order",
  "gates": { "statusReady": true, "noUncheckedOpenQuestions": true },
  "ready": true
}
```

### "Agents propose, humans ratify — mechanically enforced, not a slide."

<!--
LIVE. Land this line exactly. This is the sharpest single mechanic in
the teaser: a gate that blocks, and only a human's edit un-blocks it.
-->

---

## 3. The `--symlink` proof

**LIVE** — switch to the demo clone at `main` HEAD (real Kotlin exists there)

```
npx -y em-sdd-bridge@0.4.0 place-order --symlink
ls -la specs/001-place-order/
```

```
lrwxr-xr-x  spec.md -> ../../slices/place-order.md
```

<!--
LIVE. Trimmed to the reveal only — no spec-kit /plan or /tasks
follow-through here.
-->

---

## 3. Nothing generated, nothing duplicated

> "That's not a generated file — it's a symlink. Spec-kit is about to
> run its own `plan`/`tasks`/`implement` phases against the ratified
> doc itself."

<!--
This is the differentiation mechanic the teaser leads with elsewhere:
a spec IS a slice. No paraphrase, no second file to drift.
-->

---

## 4. A real drift, caught

**PRE-STAGED, disclosed as staged**

A real PR shipped a 24-hour cancellation grace window before the doc
caught up. The first conform run on this repo found it, cited to the
exact file and line:

```
CancelOrderCommandHandler.kt:44-47 — Duration.ofHours(24). The
INV-CO-1 check now allows cancellation up to 24 hours after Payment
Captured; the doc still says capture is an absolute bar.
```

A human ratified it the same day.

<!--
PRE-STAGED, real report, real commits. Skip the seeded-drift backstory
here — full version (Beats 13-15) is in the 25-minute script.
-->

---

## 4. Ratify: issue → divergence

**PRE-STAGED, disclosed as a staged illustration**

*"This next part is a staged illustration of the mechanism — the real
ratification happened as prose, not an in-model marker."*

```
em validate meridian-goods.em
→ warn: open issue on "Order Cancelled": conformance: code allows
  cancel within a 24h grace window after Payment Captured …

# ratified →

em validate meridian-goods.em
→ ok — no issues   (issue re-typed to divergence: never warns again)
```

<!--
Say the disclosure line plainly, every time this runs. Real
ratification here happened as prose (a report, a dated ruling, a doc
edit) — nothing in this repository's real history ever used em's
issue/divergence DSL markers. This is a staged illustration of the
mechanism, using the exact same finding.
-->

---

### "Ratified decisions stop being re-litigated — solved with two marker states, not a spreadsheet of exceptions."

<!--
Land this line. This is the alert-fatigue pain governance buyers care
about, solved with two marker states.
-->

---

## Close

**narration**

> "Every command you just watched was real. That's the whole loop —
> model, gate, build, conform, ratify — in five minutes. The full
> walkthrough runs it end to end."

<!--
Names the demonstrated-mechanics bar explicitly, same close beat as
the full script.
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
real spec-kit phase a document it never had to write.

<!--
Close slide. Repo pointer + the pilot invitation line, drawn from the
"early, credible, tryable" narrative beat. Don't oversell — this is an
invitation to try it, not a sales pitch.
-->
