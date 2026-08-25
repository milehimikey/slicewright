# Two loops: demand and supply

Everything in Acts I and II describes an artifact — the event model, the
ratified slice. This beat describes the machine the artifact runs in. There
are two loops, joined at exactly one point: a slice's `status` field.

**The demand loop runs on business time.** A need arrives — a ticket, a
stakeholder ask, a support call. A facilitated modeling session turns it
into a model delta: add a slice, change a slice, remove a slice. Nothing
enters the model until a human ratifies it. That's the loop's whole job:
turn "we should probably" into a decision with a name attached.

**The supply loop runs on technology time.** A slice that reaches
`ready-to-implement` flows out through the SDD machinery — spec, plan,
tasks, implement — into merged code. The loop closes with `em conform`:
compare the running system against the model on a cadence and report what
doesn't match, with cited evidence, for a human to rule on.

The two loops never touch except through the slice's own frontmatter. That's
deliberate — it's the only handoff surface either side needs, and it's the
one em makes into an exit code.

## The ratification gate, as an exit code

`em validate --slice-ready <key>` is not a linter warning. It's a gate that
refuses to certify a slice until four named conditions all hold, and it says
which ones don't:

```json
{
  "sliceKey": "capture-payment",
  "gates": {
    "docBound": true,
    "frontmatterUsable": true,
    "statusReady": false,
    "noUncheckedOpenQuestions": false
  },
  "ready": false,
  "diagnostics": [
    { "code": "slice-ready-status-not-ready", "...": "..." },
    { "code": "slice-ready-open-questions-unchecked", "...": "..." }
  ]
}
```

Exit 1. Flip `status: draft` to `ready-to-implement` and check the last open
question box, and the same command against the same slice returns:

```json
{
  "sliceKey": "capture-payment",
  "gates": {
    "docBound": true,
    "frontmatterUsable": true,
    "statusReady": true,
    "noUncheckedOpenQuestions": true
  },
  "ready": true,
  "diagnostics": []
}
```

Exit 0. Every slice in Meridian Goods passed this same gate before its own
implementation began — `place-order`, checked out at the tag
`walkthrough/01-model-ratified`, returns `ready: true` on all four gates
with an empty diagnostics array, the same shape as above. `em-sdd-bridge`
doesn't reimplement this check; it calls this exact command as its own
per-key precondition (`assertSliceReady`) before it will emit or symlink
anything. There is no second, looser readiness check anywhere downstream —
one gate, one place it lives, one exit code either side of it.

The gate runs in the other direction too. `em slice mark-implemented` is
idempotent — call it twice with the same PR URL and the second call is a
no-op — but it refuses a *different* URL outright:

```
$ em slice mark-implemented meridian-goods.em cancel-order https://.../pull/17
already marked implemented with a different URL
(existing: .../pull/15, requested: .../pull/17) — refusing to overwrite
```

That refusal isn't an edge case found by accident; it's the mechanism doing
its job — a slice's implementation provenance is a fact, not a field you
overwrite because a later PR happened to touch the same code.

## Deterministic core, agentic edges

Every gate, diff, and validation above is an ordinary CLI command: no
network call, no model inference, byte-identical output for byte-identical
input. That's a design ban, not a marketing line — nothing in `em`'s CLI
calls an LLM. The one place intelligence enters is at the edges: a
facilitation session proposing slice language, an implementing agent writing
code against a ratified contract, a conform run drafting a finding for a
human to judge.

The same deterministic core is now reachable two ways. `em`'s npm package
ships a second bin, `em-mcp`, exposing seven read-only tools
(`validate`, `slice_ready`, `list_markers`, `export_model`, `export_slice`,
`coverage`, `contract`) over stdio — not a separate product, the same
package, same version, same JSON builders underneath. Calling `slice_ready`
over MCP and calling `validate --slice-ready --json` over the CLI on the
same model, same moment, produces byte-identical output (modulo a trailing
newline the CLI adds and MCP's transport doesn't). One schema, two doors in.

## Agents propose, humans ratify, machines check

That's the whole division of labor, and it isn't a policy document — it's
structural. An agent can draft a slice, flag a conformance finding, or write
code against a contract. It cannot flip `status` to `ready-to-implement`,
cannot silently rewrite the model to make a conform finding disappear, and
cannot merge its own implementation without the human step that marks it. A
finding proposes; a human resolves it into either a fixed slice or a visible
`divergence`. The next beat walks that exact loop, end to end, on a real
repository — including the one time it caught something nobody had ratified
yet.
