# A ratified slice IS the spec

Every camp in spec-driven development agrees on what a spec must contain
to be worth anything: user scenarios, acceptance criteria in given/when/then
form, functional requirements, and the entities involved — precise enough
that an agent can implement without a follow-up conversation. Hold that
list, and look at what a fully-defined slice in Meridian Goods already
contains, unprompted, because it was ratified as a slice, not written as a
spec.

`slices/place-order.md`'s command table is the field-by-field contract:
`orderId` (client-generated, doubles as the idempotency key), `customerId`,
`lineItems`, `totalCents` — each with type, required-ness, and the rule that
governs it. Its invariants are numbered and each carries a rejection
outcome: `INV-PO-1` (a repeated `orderId` with the same payload is a silent
no-op; a different payload is a rejected conflict), `INV-PO-2` (`totalCents`
must equal the sum of line items — the server recomputes it and never trusts
the client's number), `INV-PO-3` (no empty orders), `INV-PO-4` (no zero or
negative quantities). Its scenarios are Given/When/Then, one happy path and
one rejection per invariant, plus the two idempotent-retry cases INV-PO-1
actually needs. Its Open Questions section carries two resolved items, each
recorded as a decision, not silently assumed. Nothing in that document was
written to satisfy a spec template. It was ratified at the model, because
that's what a fully-defined slice is.

## The mechanical proof

`em-sdd-bridge` doesn't render a copy of that document into a new spec
file and hope the two stay in sync. In `--symlink` mode it deletes
whatever spec.md would have been and replaces it with a real filesystem
symlink to the slice doc itself — there is no second file to fall out of
sync with, because there is no second file. Run it against a real,
ratified slice in Meridian Goods:

```
$ em validate meridian-goods.em --slice-ready cancel-order --json
{
  "sliceKey": "cancel-order",
  "gates": { "docBound": true, "frontmatterUsable": true,
             "statusReady": true, "noUncheckedOpenQuestions": true },
  "ready": true, "diagnostics": []
}

$ npx -y em-sdd-bridge@0.4.0 cancel-order --symlink
Linked .../specs/001-cancel-order/spec.md -> ../../slices/cancel-order.md on branch 001-cancel-order
```

And check the filesystem, not the tool's own claim about what it did:

```
$ ls -la specs/001-cancel-order/
lrwxr-xr-x@ 1 mikey  staff  28 ... spec.md -> ../../slices/cancel-order.md
$ readlink specs/001-cancel-order/spec.md
../../slices/cancel-order.md
```

That's a real symlink, on a real branch, pointing at the same document a
human ratified. spec-kit's own downstream phases — plan, tasks, implement —
run against it unmodified, because from their side it's just a file at the
path they expect.

The bridge won't do this for a slice nobody has signed off on. Try it
against an unratified slice — `status: draft`, or an unchecked open
question — and `assertSliceReady` refuses before it ever touches the
filesystem, citing exactly which of the four gates failed. It doesn't
reimplement that check; it calls the same `--slice-ready` command shown
above as its own precondition. There is one readiness gate, in one place,
and every downstream tool defers to it.

## The invariant carries its own number through every layer

Where the bridge does render output — the non-`--symlink`, emission path,
for tools that need a real file — every generated functional requirement
carries a literal back-reference to the invariant it came from:

```
FR-003 ... At least one entry (INV-PO-3)
FR-004 ... INV-PO-2. The server recomputes and verifies this...
```

`INV-PO-2` is one token, greppable from the slice doc through the generated
requirement to whichever test asserts the rejection. When the business
changes the rule, that's the exact set of artifacts that changes with it —
nothing else, because nothing else claims to derive from it.

## Takeaway

Spec-writing disappears as a separate activity, not because the discipline
of writing a spec was wrong, but because the discipline was already being
done — at the model, under a different name, with a human's signature on
it before an agent ever touched the code. Ratification replaces
specification; the file on disk, symlink or generated copy, is just how the
already-ratified decision reaches the tool that needs to read it.
