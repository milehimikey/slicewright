# Every change is add, change, or remove a slice

Once a system is live, every change it will ever undergo is one of three
operations on its definition: add a slice, change a slice, remove a slice.
A new feature is new commands, views, or reactions — added slices. A
changed business rule is a changed invariant, contract, or scenario on an
existing slice. A retired capability is a removed slice. That's the whole
change-management model, and the honest fine print is that it governs the
*definition* of the system, not every line of engineering work — a refactor
that changes none of the above isn't a model change at all, it's the supply
loop's private business.

This matters because it means "what changed and who approved it" stops
being an archaeology project and becomes a query a tool can answer. Here's
Meridian Goods proving it, on a real change: self-service order
cancellation, added months after the base ordering flow shipped.

The change lands as two new slices — `Cancel Order` (a State Change) and a
repeated instance of the existing `Open Orders` view, now folding in
`Order Cancelled` — authored, ratified, and reviewed on their own branch
before a line of Kotlin exists:

```
$ em diff meridian-goods.em --from HEAD
2 slices added

+ slice "Cancel Order"
+ slice "Open Orders — cancelled"
```

That's not a text diff of a markdown file — it's a structural diff of the
model, reviewed as operations. The JSON form carries the same shape a CI
check or a reviewer's tool would read:

```
$ em diff meridian-goods.em --from HEAD~1 --json
{
  "diffSchemaVersion": "1.6",
  "generator": { "name": "@milehimikey/em", "version": "1.8.0" },
  "counts": { "slicesAdded": 2, "slicesRemoved": 0 },
  "changes": [
    { "type": "slice-added", "name": "Cancel Order", "sliceKey": "cancel-order" },
    { "type": "slice-added", "name": "Open Orders — cancelled", "sliceKey": "open-orders-cancelled" }
  ]
}
```

And once the change is committed, it renders as a dated entry in the
model's own history — the artifact for the stakeholder who will never open
a diagram:

```
## 2026-08-23 — Change beat: Cancel Order + repeated Open Orders view (model + docs) (e779952)

2 slices added

+ slice "Cancel Order"
+ slice "Open Orders — cancelled"
```

## The repeated view is not a new relationship

`Open Orders — cancelled` is the same read model reappearing later on the
timeline, not a second view wired backward to the first. Two `Open Orders`
instances share a name; no arrow connects them, and no arrow ever will —
each is a separate render of the same logical view at a later point in the
story, and the diagram shows exactly that: two columns, no edge between
them. That's what lets a view evolve for years without ever becoming a
retroactive edit to something already shipped.

## Cancel is not free of judgment, and the model says so

`Cancel Order` needed two real decisions, not just a new command. Should a
repeated cancel on an already-cancelled order be a rejection or a no-op?
Meridian Goods' answer — a no-op, because a cancel carries no data that
could conflict with an earlier cancel, unlike `Place Order`'s idempotency
case — is recorded as `INV-CO-2`, in the slice doc, not left to whoever
implements it first to decide quietly. Should a cancelled order disappear
from the fulfillment view or stay visible with a status flag? The model
picked removal, and says so as `INV-OOC-1`, with its reasoning attached: a
cancelled order has no further fulfillment action, and a view that wants
cancelled rows visible for history is a distinct, unmodeled read model. Both
calls are the kind of decision a backlog item usually buries in whoever
wrote the code — here they're dated, ratified, and grep-able.

## Definitions and work items are different things

The branch that carried this change — `change-cancel-order`, one PR, not
merged into the demo repo's history for this beat's purposes — is scaffolding.
It gets created, reviewed, and discarded. The two slice documents it
produced are not scaffolding; they live in the model for the life of the
system, versioned in place, the same way `place-order.md` has lived there
since the base model shipped. That split is what lets "what changed" stay a
one-command answer months or years later: `em diff` between any two
revisions, and `em changelog` for the human-readable version, work exactly
the same whether the change happened yesterday or eighteen months ago.

## Takeaway

"What changed and who approved it" becomes a query, not an archaeology
project — because the only thing that ever changes is a slice, in one of
exactly three ways, each one a structural diff and a dated ledger entry the
day it happens. That's Act I and Act II's whole claim, demonstrated on the
same repository this story keeps coming back to. What makes that claim
trustworthy at scale — the two loops that ratify a change and check it
against reality forever after — is Act III.
