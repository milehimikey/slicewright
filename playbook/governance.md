# Governance

This is for your engineering lead and whoever in your org owns audit or
compliance sign-off for a new tool touching the spec-to-code path. By the
end of it you will know exactly what to hand an architecture review board
before the pilot starts, what the audit trail looks like once it's
running, and a mapping from the control questions a reviewer actually
asks to the specific artifact or command that answers each one.

[narrative/08](../narrative/08-governance-without-ripping-out.md) makes
the case for four governance pillars — human-ratification gates,
evidence-cited drift reports, decision lifecycle and lineage, and a
deterministic core — each backed by shipped mechanism, not a policy
document. This doc extends those four pillars into the procedure your
org's existing review process actually needs: what to hand a board in
advance, what the trail looks like in production, and where the honest
limits are.

## What to hand an architecture review board before the pilot

Four artifacts answer the four questions a review board asks before
anything ships:

**"Can an agent ship scope nobody signed off on?"** — `em validate
<model> --slice-ready <key>` is the readiness gate: it exits non-zero
unless four named checks all pass (`docBound`, `frontmatterUsable`,
`statusReady`, `noUncheckedOpenQuestions` — add `--json` to see each one
individually rather than parsing stderr prose). `em-sdd-bridge` delegates
to this exact command as its own precondition instead of reimplementing
the check, and the implementing agent's own written contract requires
gating on it before touching code. Hand the board the gate's exit-code
contract, not a description of intended behavior — it's a command they
can run themselves against your model.

**"Who approved this, and can we prove it?"** — `em slice ratify <model>
<key> --by "<name>"` writes `ratifiedBy:`/`ratifiedOn:` directly into the
slice doc's own frontmatter, and refuses to silently overwrite a
different name already recorded there. That's the named-approver record:
not a step in a process document, a field in the artifact itself,
queryable in git history (see [metrics.md](metrics.md)'s ratification
turnaround procedure for the exact command pattern).

**"Is the approval routed through review, or can anyone just run the
command?"** — a `CODEOWNERS` entry on `slices/**`, with "Require review
from Code Owners" enabled on branch protection, routes every ratification
through a designated reviewer before the PR can merge — the same
enforcement your repo already leans on for any other reviewed path. This
is a convention `em` documents, not something `em` itself checks; the
routing lives entirely in your git host's branch protection.

**"Is the tool itself something we now have to audit?"** — no LLM call
exists anywhere inside `em`'s CLI, and its output is byte-deterministic
for identical input: the same command run twice produces identical
output, and the CLI and the MCP server reading the same model at the same
moment produce identical output too. A reviewer can trust what the tool
proved without trusting any model's inference, and there's no hosted
service or vendor dependency to add to the org's own vendor-risk review —
plain text, git-native, works offline.

## What the audit trail looks like during and after

**The business-readable ledger.** `em changelog <model>` renders the
model's own git history as one section per commit that touched it,
newest first, each with its structural delta and any dated decisions
woven in from the project's state file — a reviewer who will never open
a diagram can read this and understand what changed, when, and (where a
decision was logged) why.

**Dated compliance evidence.** Each `conformance/<date>-report.md` file
is a dated, evidence-cited check of the running system against the
ratified model — file:line citations, a classification per finding, a
`Ratification record` section naming who ruled on each finding and how
("model bends" vs "code bends"), dated. That in-file ratification record
is the compliance evidence itself, not a summary of it.

**The correction mechanism that never rewrites history.** A ruled report
is a snapshot; the model keeps moving after it. `em conform-supersede
<model> <report-path> --as-of <rev> --findings <spec>` stamps a ruled
report with a "superseded as of `<rev>`" banner the moment its findings
are ratified — an additive splice inserted directly under the report's
title, never a rewrite. Every byte of the report after that line survives
untouched, so a reader who finds an old report later sees immediately
that its citations describe an ancestor of the current model, and the
report itself stays real, unaltered history.

**The continuous-assurance signal.** `em status <model> --tests <dir>`
gives one deterministic rollup — slices by lifecycle status, invariant
coverage, open issues, and a conformance clause reading "last conformed
`<rev>` — N commits and M slice-PRs behind HEAD." `em freshness <model>`
is the same conformance fact standalone, for when that's all a check
needs. Neither one is a gate by itself; both are the signal that tells a
reviewer, or a scheduled job, whether the system is due another conform
cycle.

**The version/content-agreement control, in CI.** `em ledger <model>
--from <rev> [--to <rev>]` checks that a slice doc's `version:` field and
its actual content — body text plus lineage fields — always change
together between two revisions: a content change with no version bump is
a stale ratification signal, a version bump with no content change is a
no-op, and a version going backwards is almost always a typo. It's
opt-in and CI-recipe-tier, wired by `em ci init`'s generated
`em-ci.yml`, deliberately never folded into the fast, git-history-free
`em validate` check.

Conform itself stays advisory throughout: `em ci init` also scaffolds
`em-conform.yml`, a weekly (or manually dispatched) advisory run that
never fails the build and never applies anything — findings land in a
report or an issue, and only a human ratifying them and running `em
state set-conformance` ever advances the record. Nothing above describes
drift as auto-applied or CI-blocking, because nothing in the design is.

## Control questions, mapped to artifact and command

| Control question | Artifact / command |
|---|---|
| Who approved this scope, and when? | `ratifiedBy:`/`ratifiedOn:` in the slice doc's frontmatter, written by `em slice ratify --by`, routed through the `CODEOWNERS` entry on `slices/**` |
| When was the system last verified against its own spec? | `em freshness <model>` / `em status <model> --json`'s `conformance` clause — "last conformed `<rev>` — N commits and M slice-PRs behind HEAD" |
| How are exceptions recorded, and do they expire? | the `issue "..."` → `divergence "..."` lifecycle on the diagram, plus each conformance report's own `Ratification record` section (disposition per finding, dated). Divergences don't auto-expire: a ratified divergence persists as dated record until a later ruling re-types or removes it — expiry is a re-ratification, never a timeout |
| What changed, and why? | `em changelog <model>` — the model's git history rendered as a business-readable ledger, decisions woven in by date |
| Is the model's own version bookkeeping trustworthy? | `em ledger <model> --from <rev>`, run in CI via `em ci init`'s generated workflow |
| Is a stale report still being cited as current? | the `em conform-supersede` banner — if a report has one, its citations describe an ancestor of the current model |
| Can we trust the tool's own output without auditing the tool? | the deterministic-core property: no LLM in the CLI, byte-identical output between two runs and between the CLI and MCP |

## Honest limits

Mirroring [narrative/08](../narrative/08-governance-without-ripping-out.md)'s
own concessions, stated at the same size here:

The tooling cannot force incremental ratification. Nothing stops a team
from batching months of modeling before writing anything down — the
mechanism only makes batching visibly costly, since an unratified
backlog shows up as a pile of red `issue` notes nobody's resolved.
Neither can it stop someone editing a slice doc's YAML by hand outside
`CODEOWNERS` routing: the routing lives in your git host's branch
protection, not in `em` itself, and nothing in a plain-text, git-native
tool can distinguish "`ratifiedBy` was set by running `em slice ratify`"
from "someone typed the same field by hand."

The conformance loop's real-world evidence is two dated runs, not a
track record. On Meridian Goods, the stable fact is 20 distinct
invariants, 0 uncovered — the raw per-version count the tool prints has
moved as coverage-attribution bugs were fixed, which is exactly why this
playbook and the narrative both cite the distinct count, never the raw
one. A pilot's own conformance reports are the next data points, not a
replacement for the two that exist — see [metrics.md](metrics.md) for
what a pilot commits to recording as it runs.

For the week-by-week mechanics of when to run the readiness gate, when to
ratify, and when a conform cycle happens during the pilot itself, see
[week-by-week.md](week-by-week.md). For who on your team owns each of
these steps — including the conform-cadence owner referenced throughout
this document — see [roles.md](roles.md).
