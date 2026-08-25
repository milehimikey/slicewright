# The Meridian Goods walkthrough

Everything in the last six beats is a claim about a methodology. This one
points at a repository instead of arguing: `meridian-goods`, a small
ecommerce ordering system, modeled fresh in the merged slice shape, fully
implemented in Kotlin against Axon Framework 5, and run through one
complete conform-and-ratify cycle. Every command in the locked walkthrough
script reproduces from a clean clone of that repository; the stages are
checkpointed as seven `walkthrough/*` tags against real commits, not staged
screenshots.

`em` itself ships a bundled model-only sample in a similar domain — an
ecommerce checkout-and-fulfillment flow, seven slices, no code. We're not
pretending Meridian Goods occupies different territory; we're showing what
depth looks like once a model has code behind it: ratified slice docs,
INV-cited tests, a conform run with findings a human actually ruled on. The
sample proves the shape parses. Meridian Goods proves the loop runs.

## The model

Eight slices, four patterns, no exceptions:

```
Place Order            — State Change  (Customer → Ordering)
Payments To Request     — the automation's read model
Request Payment         — Automation (reaction + command + event, one slice)
Record Payment Result   — Translation (payment-provider webhook)
Open Orders             — State View (staff-facing)
Order Status            — State View (customer-facing)
Cancel Order            — State Change (the change beat)
Open Orders — cancelled — State View (a repeated instance — no arrow back to the first)
```

`Place Order` is the deep-dive slice: client-generated `orderId` for
idempotency, `totalCents` recomputed server-side rather than trusted from
the client, four named invariants (`INV-PO-1` through `INV-PO-4`). Its
slice doc is what `em-sdd-bridge --symlink` hands to spec-kit:

```
$ npx -y em-sdd-bridge@0.4.0 place-order --symlink
```

The bridge deletes the rendered `spec.md` it would otherwise generate and
replaces it with a real filesystem symlink to `slices/place-order.md`. Every
generated functional requirement carries a literal back-reference to the
invariant it came from — `FR-002 ... INV-PO-2. The server recomputes and
verifies this` — so spec-kit's own `plan`/`tasks`/`implement` phases run
against the ratified slice itself, not a paraphrase of it.

## The change beat

`Cancel Order` is the walkthrough's change: a real feature request — let
customers self-cancel from the Account Page — modeled as an added slice,
not a code-first patch. It reads `Order Placed` and (for one invariant)
`Payment Captured`, and it feeds a *second*, independent instance of
`Open Orders` fed by `Order Cancelled`. That second instance takes no arrow
back to the first — repeated views never connect to each other, they're
each their own fold from their own events, and the model says so by
construction.

## The drift, caught

Here's the exhibit worth reading closely, because it's not staged. PR #17
shipped a real behavior change to `Cancel Order` — a 24-hour grace window
after payment capture, so a customer who cancels right after a charge lands
doesn't need a manual refund — before anyone touched the model or the doc.
The first conform run on this repository found it:

> `CancelOrderCommandHandler.kt:44-47` — `val CANCELLATION_GRACE: Duration =
> Duration.ofHours(24)`. The INV-CO-1 check now allows cancellation up to 24
> hours after `Payment Captured`. Both `slices/cancel-order.md`'s INV-CO-1
> text and `domain-decisions.md`'s cancellation bullet still say capture is
> an absolute, exception-free bar. Two independent doc sources agree with
> each other and disagree with the code.

That's one of four findings from the run — one seeded by the team running
the demo, three organic, nobody planted them. Zero false positives across
all four; `em diff`'s structural surface came back `"identical": true`
because every field and every flow still matched the model exactly — the
drift was entirely behavioral, not structural. The finding was proposed as
an `issue`, never auto-applied. The ratifier read it the same day and ruled
the behavior intentional: fix the doc, not the code. `slices/cancel-order.md`
now carries this:

```
## Delta
- MODIFIED (v1 → v2, ratified 2026-08-23): INV-CO-1 — "cancel only before
  Payment Captured" became "cancel before capture, or within the 24h
  post-capture grace window." Origin: shipped in PR #17 ahead of the model;
  caught by the conformance run; ratified as intentional. Implementation
  already conforms — no propagation work.
```

`version: 2`, a re-typed invariant, a dated ratification record, and a
`Delta` block a future reader can find without opening git. The other three
findings that same day: one process reworked because a doc-described
mechanism had quietly diverged from what shipped, one rejection reworded
into an actual rejection, one stale generated index re-run. All four ruled
in one sitting, same day as the report.

## What's checkpointed

Seven tags, each on a real commit: model ratified, first slice implemented,
all base slices implemented, the change beat landed, the drift shipped, the
conformance report opened, the ratification merged. Check any of them out
and every command in the script — render, validate, the readiness gate,
the symlink, `em diff` between two of them, `em coverage --strict` — runs
exactly as scripted, because that's what "dry-run every beat before
recording it" means.
