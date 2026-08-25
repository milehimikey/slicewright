# Describe the business process, not the intended code

An event model describes a system as a timeline. Left to right is time; along
it runs the story of how the business actually works. Everything on the
timeline is one of three kinds of element — events (facts, past tense:
`Order Placed`), commands (intentions, imperative: `Place Order`, which the
system may reject), and views (read models, noun phrases: `Open Orders`) —
plus a top row of screens and API calls wiring a person or client to the
command they issue or the view they're looking at. That's the whole
notation. The discipline lives in the timeline, not in a vocabulary of
symbols, which is deliberate: a stakeholder can read it in minutes.

Here is the actual model for Slicewright's own proof artifact, Meridian
Goods — an ecommerce ordering system — read straight out of its source file:

```
slice "Place Order" {
  ui Storefront @Customer
  command Place Order {
    orderId: UUID
    customerId: UUID
    lineItems: OrderLine[]
    totalCents: Int
    placedAt: DateTime
  } note "slices/place-order.md"
  event Order Placed @Ordering {
    orderId: UUID
    customerId: UUID
    lineItems: OrderLine[]
    totalCents: Int
    placedAt: DateTime
  }
}
```

That block *is* the model — plain text, versioned and diffed like code, not
a picture kept separately from a spreadsheet of requirements. Run the
validator against it:

```
$ em validate meridian-goods.em --json
{"ok": true, "summary": {"errors": 0, "warnings": 0, "total": 0}, "diagnostics": []}
```

And render it live:

```
$ em watch meridian-goods.em -o meridian-goods.svg --serve --port 4173
rendered meridian-goods.svg (99ms); http://localhost:4173/?svg=meridian-goods.svg
```

That URL serves one self-contained page: the diagram inlined as SVG, pan and
zoom on the model's own coordinate space, a Review mode for walking it
slice by slice, and a live-reload socket so an edit to the file redraws the
page without a manual refresh. No separate viewer binary, no file:// static
page with a stale copy — one served URL, always current.

## Four shapes, no fifth

Every slice on that timeline is exactly one of four patterns, and the
pattern fixes what the slice must contain: **State Change** — someone asks
the system to change, it records events or rejects with a reason (`Place
Order`, above). **State View** — past events folded into something a
screen or a reaction reads; nothing here decides anything, it's a pure
projection. **Automation** and **Translation** — the two patterns where the
system acts on its own, one triggered by its own state, the other crossing
a system boundary.

Automation is the pattern most worth getting right on paper, because it's
the one every other event-modeling account is tempted to draw as two
separate slices — a to-do-list view in one, the command it triggers in
another — bundled together only as a special case. Meridian Goods' payment
automation shows why that split is unnecessary: the reaction, the command
it issues, and the event it records live in *one* slice.

```
slice "Payments To Request" {
  view Payments To Request from "Order Placed" { orderId, totalCents, placedAt }
}

slice "Request Payment" {                  # Automation: reaction + command + event together
  processor Payment Requester from "Payments To Request"
  command Request Payment { paymentId, orderId, amountCents, requestedAt }
  event Payment Requested @Payments { paymentId, orderId, amountCents, requestedAt }
}
```

The to-do-list view (`Payments To Request` — orders that exist and haven't
had payment requested yet) still gets its own slice, because it's a
distinct read model with its own fold rule and its own consumer, the
processor. But the reaction, the command it fires, and the fact it records
are one unit: one spec, one branch, one PR, no exception carved out for
"but this pattern is different." A processor never records an event
directly — it funnels its decision through the command like any other
trigger, so the command keeps its invariants no matter who calls it — but
that discipline lives inside a single slice, not across a mandated pair of
them.

Translation works the same way for a system boundary instead of an internal
to-do list: Meridian Goods' `Record Payment Result` slice is a payment
provider's webhook, translated and recorded, in one slice, with the
adapter never permitted to write a domain event directly.

## What ties it together

Two rules keep the timeline honest, and they cost nothing to check by eye
or by tool: no event causes another event directly — a command, with its
rules, always sits between two facts — and arrows never point backward; a
view that keeps evolving as later events land reappears later on the
timeline as a new instance under the same name, rather than dragging a
connection back to itself. Nothing above is a diagram convention for its
own sake. It's what makes a slice checkable: every view traces to real
events on its left, every event has a reader on its right, every command
has something that can actually trigger it. `em validate` runs that check
in milliseconds; a stakeholder can run the same check with their eyes on
the rendered page.

## Takeaway

There is now one artifact that a CFO and a coding agent can both read —
the same file, the same render, no translation step between "what the
business does" and "what the machine will build." The next question is
what happens when that artifact has to become code without anyone writing
a second document to get there.
