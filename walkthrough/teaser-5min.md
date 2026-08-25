# The Meridian Goods walkthrough — 5-minute teaser

**Runtime**: ~5 minutes. Compresses the full script's two-loop arc
(demand: model → gate → build; supply: conform → ratify) into one pass
without the modeling build-up. Assumes the diagram already exists — no
live Socratic authoring here. Same pinned versions as the full script
(`em 1.8.0`, `em-sdd-bridge 0.4.0`) and the same real captures, trimmed
harder. Full detail, mode markers, and exact invocations for every beat
referenced below live in `script.md`; this file only tightens the timing.

---

## 1. Flash the finished diagram

**0.5 min · PRE-STAGED**

```
git checkout walkthrough/01-model-ratified
em watch meridian-goods.em -o meridian-goods.svg --serve --port 4173
```

"Here's Meridian Goods' ordering process, modeled as a timeline — six
ratified slices, four patterns, zero lines of code yet." Open the served
URL, pan across it once. Don't narrate the render mechanics; this is the
`script.md` Beat 3 artifact shown, not re-explained.

---

## 2. Gate fail → pass

**1.5 min · LIVE**

Same sequence as `script.md` Beat 6, trimmed to the two `--slice-ready`
calls and the one-line fix — skip naming all four gates individually,
just show `"ready": false` flip to `"ready": true` after a human checks a
box and flips `status`:

```
em validate meridian-goods.em --slice-ready place-order --json
→ "ready": false   ("statusReady": false, "noUncheckedOpenQuestions": false)

# human checks the box, flips status: ready-to-implement

em validate meridian-goods.em --slice-ready place-order --json
→ "ready": true    (all four gates true, empty diagnostics)
```

**Line to land**: "agents propose, humans ratify — mechanically enforced,
not a slide."

---

## 3. `--symlink` proof

**1 min · LIVE**

Switch to the demo clone at `main` HEAD (real Kotlin code exists there —
the bridge needs it). Trimmed to the reveal only, no spec-kit
`/plan`/`/tasks` follow-through:

```
npx -y em-sdd-bridge@0.4.0 place-order --symlink
ls -la specs/001-place-order/
→ lrwxr-xr-x  spec.md -> ../../slices/place-order.md
```

"That's not a generated file — it's a symlink. Spec-kit is about to run
its own `plan`/`tasks`/`implement` phases against the ratified doc
itself. Nothing generated, nothing duplicated."

---

## 4. Drift → divergence, compressed

**1.5 min · PRE-STAGED, disclosed as staged**

Skip the seeded-drift backstory (real PR #17, real conform report — full
version in `script.md` Beats 13–15); go straight to the cited finding and
the marker flip:

"A real PR shipped a 24-hour cancellation grace window before the doc
caught up. The first conform run on this repo found it, cited to the
exact file and line, and a human ratified it the same day." Show one line
from the real report:

```
CancelOrderCommandHandler.kt:44-47 — Duration.ofHours(24). The INV-CO-1
check now allows cancellation up to 24 hours after Payment Captured; the
doc still says capture is an absolute bar.
```

Then the marker mechanism (say plainly: *"this next part is a staged
illustration of the mechanism — the real ratification happened as prose,
not an in-model marker"*):

```
em validate meridian-goods.em
→ warn: open issue on "Order Cancelled": conformance: code allows cancel
  within a 24h grace window after Payment Captured …

# ratified →

em validate meridian-goods.em
→ ok — no issues   (issue re-typed to divergence: never warns again)
```

**Line to land**: "ratified decisions stop being re-litigated — solved
with two marker states, not a spreadsheet of exceptions."

---

## 5. Close

**0.25 min · narration**

"Every command you just watched was real. That's the whole loop — model,
gate, build, conform, ratify — in five minutes. The full walkthrough runs
it end to end."

---

**Total**: ≈ 5 min including transitions. This is the smallest arc that
still proves both halves of "agents propose, humans ratify" — a gate that
blocks, and a drift report only a human resolves — plus the sharpest
single differentiation mechanic (`--symlink`).
