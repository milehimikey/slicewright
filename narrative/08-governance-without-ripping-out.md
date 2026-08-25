# Governance without ripping anything out

An enterprise architecture review board doesn't ask "does the model look
nice." It asks who approved this, whether an agent can silently rewrite the
system of record, whether there's an audit trail, and whether the tool
itself is something they now have to audit too. Four pillars, each one
shipped mechanism answering one of those fears — not a policy document
promising good behavior, a thing that already refuses to misbehave.

## Pillar 1 — Human-ratification gates

**The fear**: an agent ships scope nobody signed off on.

**The mechanism**: `em validate --slice-ready` (Beat 6) is not a
convention an agent could route around — `em-sdd-bridge` delegates to this
exact command as its own precondition rather than reimplementing the
check, and the implementing agent's own written contract requires gating on
it before touching code. On Meridian Goods, this gate ran on all eight
slices before a line of Kotlin existed. `docs/process.md`'s maxim: "Agents
propose. Humans ratify."

## Pillar 2 — Evidence-cited drift reports, never auto-applied

**The fear**: an AI silently rewrites your system of record based on its
own inference.

**The mechanism**: `em conform` produces a structured report — file and
line citations, a classification per finding (real drift / model gap /
internal inconsistency / uncertainty) — that proposes, never applies. The
report Beat 7 walked through is the real artifact: four findings, each with
a `Proposed red note` a human either ratifies or rejects, and a line at the
bottom of the file that reads, honestly, "none yet — awaiting
ratification" until a human actually rules. Nothing in the loop writes to
the ratified model without that step.

## Pillar 3 — Decision lifecycle and lineage

**The fear**: how do we know a decision was actually decided, by whom, and
why — and how do we stop re-litigating it every audit cycle?

**The mechanism**: `issue "..."` renders a red open-question marker
directly on the diagram; once ratified, it re-types to `divergence "..."`
(teal), which never raises a future validate warning again. Every slice doc
carries required frontmatter (`status`, `version`) plus lineage keys
(`split-from`, `merged-from`, `superseded-by`) so a real split reads as
"split from," never silent remove-and-add. `cancel-order.md`'s jump from
`version: 1` to `version: 2` with a `Delta` block *is* this pillar,
concretely: a reader who opens that file six months from now doesn't need
git archaeology to learn that the rule changed, when, and why. `em
changelog` renders the same history as a business-readable ledger, newest
first, for the reader who will never open a diagram at all.

## Pillar 4 — Deterministic core, agentic edges

**The fear**: is the tool itself trustworthy, or is it another
probabilistic black box we now have to audit?

**The mechanism**: no LLM call anywhere inside `em`'s CLI; byte-deterministic
output for identical input, confirmed the same way twice — once between two
runs of the same command, once between the CLI and the MCP server reading
the same model at the same moment. An auditor can trust what the tool
proved without trusting any model's output, and there's no hosted service
or model vendor to depend on: plain text, git-native, works offline.

## Interface, don't adopt

None of this asks anyone to replace spec-kit, Kiro, or whatever SDD tool an
org has already standardized on. `em-sdd-bridge` replaces exactly one step
— `/speckit.specify` — either by symlinking straight to the ratified slice
doc or, where a filesystem symlink doesn't fit the workflow, by rendering an
equivalent file from it. `plan`, `tasks`, and `implement` run unmodified.
The stance: the model owns the specification source; the SDD tool keeps its
conventions and its network effects; nobody has to migrate anything to get
the ratification gate and the conform loop working underneath what they
already run.

## Where this is honest about its limits

Two-loop architecture is not phase-gating — slices ratify incrementally, one
at a time, not in a single upfront phase — but that only holds if a team
actually keeps ratifying incrementally instead of batching months of
modeling before writing anything down. The tooling can't force that
discipline; it can only make batching visibly costly, since an unratified
backlog shows up as a pile of red `issue` notes nobody's resolved.

The conformance loop's real-world evidence is two dated runs, not a track
record — the next beat states both numbers exactly, because a methodology
that ships a drift-checker had better not drift from its own claims. And
"interface, don't adopt" has been validated live against exactly one SDD
tool, spec-kit. It's a reasonable bet that the same thin-adapter pattern
holds for others; it isn't a demonstrated fact for any tool but the one
that's actually been run.
