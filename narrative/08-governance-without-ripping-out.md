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

That maxim used to name a step, not a person — a passing gate said *a*
human ratified, not *which* one. `em slice ratify --by "<name>"` closes
that gap: it flips `status` to `ready-to-implement` and records
`ratifiedBy`/`ratifiedOn` directly in the slice doc's own frontmatter, and
it refuses to silently overwrite a different name already recorded there,
the same discipline `mark-implemented` already holds for a PR URL. A
documented `CODEOWNERS` entry on `slices/**` routes that flip through a
designated ratifier's review before it can merge at all. Neither piece
stops someone editing the YAML by hand outside that routing — nothing in
a plain-text, git-native tool can — but the *intended* path is now the one
that also leaves a name and a date, not just a timestamp on a commit.

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

A ruled report is still a snapshot, and a snapshot left unlabeled starts
lying the moment the next commit lands — Beat 7's own citations are a
case in point. `em conform-supersede` closes that gap by stamping a ruled
report with a "superseded as of `<revision>`" banner the moment its
findings are ratified, so a reader who finds it later — a search hit, a
link from an old PR — sees immediately that its file:line citations
describe an ancestor of the current model, not its current state. It's an
additive splice, never a rewrite: every byte of the report after that one
inserted line survives untouched, so the report stays real history, not a
document quietly edited into looking current.

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

## The state-of-the-system line

None of the four pillars above answer the question a reader actually opens
this repo to ask: **is this system healthy, right now?** Before this
release that answer lived nowhere as a single fact — it had to be
assembled by hand from a README table, a conformance report, and a state
file, at least one of which is hand-maintained and can go stale without
anyone noticing (Meridian Goods' own generated Slices table did exactly
that — a real, dated finding, not a hypothetical). `em status` closes
that gap with one deterministic line, and it isn't hypothetical either —
it's the real output on Meridian Goods, run against this release ahead of
its own launch:

```
8/8 implemented · 20/20 invariants covered · 0 open issues · last conformed
8f12ed8 — 11 commits and 0 slice-PRs behind HEAD
```

Same posture as everything else in this story: no LLM, byte-deterministic
for identical input, and it renders as a markdown block or an SVG badge a
README can embed instead of a paragraph someone has to keep rewriting by
hand — this exact repository now carries both. `em ci init` does the same
kind of work for the CI side of the cookbook this beat already promised:
validate, the readiness gate, coverage, a badge rebuild, an advisory
conform cadence — from a page of YAML a team used to copy-paste by hand to
one command that installs it, marker-delimited so a team's own additions
survive the next `em` upgrade.

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
