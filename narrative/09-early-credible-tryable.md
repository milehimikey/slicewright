# Early, credible, tryable today

Slicewright is MIT-licensed, on npm, with zero hosted dependencies. One
package, `@milehimikey/em`, ships the CLI, the facilitation skill, and the
MCP server together — the same `bin` block carries both entry points:

```json
"bin": { "em": "./dist/cli.js", "em-mcp": "./dist/mcp/main.js" }
```

There's no separate account to create, no service to trust with your model,
nothing to self-host. `git clone`, `npm install`, `em init` — everything
that follows in this story ran on exactly that.

## What's demonstrated, what's argued, stated at its own size

The claims register behind this story holds 18 claims: 10 demonstrable
today, 5 backed by exactly one or two dated data points, 3 argued from
principle and labeled as argument. None of them are overstated past their
class, and the register is the thing an editor checks this narrative
against, not the other way around.

The claim most worth being precise about is the conformance loop's track
record, because it's the one an audience is right to be skeptical of. It is
now **two** dated data points, not one:

| Date | Codebase | Slices | Invariants | Findings | False positives |
|---|---|---|---|---|---|
| 2026-07 | (first validated run) | 19 | 58 | 10, incl. real drift | 0 |
| 2026-08 | Meridian Goods | 8 | 20* | 4 — 1 seeded, 3 organic | 0 |

\*20 distinct invariants, 0 uncovered — the tool's raw attribution count
varies by version (25 on em 1.8.0, 24 on 1.8.1, 20 on this release, which
fixed a wrapped-continuation-line overcount). This release is the first
where the raw count and the distinct count agree; the distinct count was
always the stable fact, and now it's also the number the tool prints.

(The 2026-07 cycle also included a separate controlled check that seeded 3
known drifts; the loop caught 3 of 3.) Both runs caught real drift nobody
planted and produced zero false positives. Both were ratified same-cycle,
with dated rulings, not left as open findings. That is real, and it's worth saying
plainly — but it is two data points, not a track record, and the honest
version of this claim will keep saying "two runs, dated" for as long as
that's true, and will update the moment it isn't. A methodology that ships
a drift-checker and then oversells its own evidence has disqualified
itself by its own standard.

One more claim earned its evidence late enough to state exactly: the
implementation contract's agent-neutral path has been **exercised by an
agent without the bundled skill** — discovery via AGENTS.md, the contract
via `em contract`, reads via `em export --slice` and the MCP server (whose
outputs matched the CLI byte-for-byte) — which deleted and re-implemented a
slice to a green suite with every invariant re-cited. That is one controlled
exercise, by one agent, still from the same vendor family; a cross-vendor
run is an open invitation, not a claimed result.

The same discipline applies everywhere else in the register. The 30-slice
field record where no generated spec was ever hand-edited after generation
is one data point, dated. The bridge's live validation against spec-kit
covers exactly one SDD tool, for two real slices — Kiro is a documented,
honest gap, not a quiet omission. The claim that structural completeness
checking beats prose backlogs at catching whole classes of drift is
argued from mechanism, not measured in an A/B test, and it's labeled that
way every time it appears.

## The toolchain is growing; here's exactly how far each piece got

Everything above describes `em` and `em-sdd-bridge`, both shipped and both
exercised on Meridian Goods. This release adds two more pieces, and this
story states their status at the same size it states everything else's:
*decided and prototyped* is not the same claim as *shipped and run*.

**`em-portal`** is a separate package, version 0.1.0, not yet on npm — a
fully static site over `em export`/`em status` JSON meant to give a
non-technical stakeholder a guided first read of a team's own model
instead of a generic legend, a status line instead of a slice list to
scroll, and navigation across more than one model. The decision to build
it as a separate tool rather than reworking `em catalog` is made and
written down, and a throwaway prototype benchmarked the approach up to
1,219 slices across 20 models (sub-100ms, in-process) to confirm the
`em export`/`em status` substrate isn't the bottleneck at that scale. The
0.1.0 scaffold itself — CLI, a status-first landing page, per-model and
per-slice pages, deterministic builds — exists as real, tested code; the
guided-teaching layer and async review intake are later, still-unbuilt
milestones on the same project. No pilot has opened any of it yet.

**`em-tracker-bridge`** is also 0.1.0, also in development, with no public
repository yet — the same thin-adapter shape `em-sdd-bridge` already
proved for spec-kit, aimed at an org's issue tracker instead: mirroring a
slice's lifecycle status (ratified, implemented, drift found) into Linear
first, Jira later, without `em` itself becoming a tracker. It exists today
as a scoped project with milestones, not as code that has synced a single
real slice to a real tracker.

Both are named here for the same reason the conformance loop's evidence is
stated as "two runs, dated" instead of "proven": a reader deciding whether
to pilot this toolset deserves to know which pieces have a dated run
behind them and which ones are still a decision record and a prototype.
The piece of the surrounding machinery that *has* shipped and run — the
`em status` line and the conformance-report freshness stamp, both
exercised for real on Meridian Goods ahead of this release — is Beat 08's
evidence, not this one's; this beat only tracks the parts still short of
that bar.

## What we're asking for

This is a pilot invitation, not a product launch with a sales team behind
it. If your team already runs an SDD tool, the ask is small: point the
bridge at one slice, watch it refuse an unratified one, watch `--symlink`
hand a real spec-kit phase a document it never had to write. If you don't,
the whole loop — model, gate, implement, conform, ratify — is the complete
pipeline; there's nothing else to install underneath it.

What we intend to collect from a real pilot, honestly stated in advance
rather than promised vaguely: conform-cycle cadence and finding counts per
codebase, ratification turnaround time, how often a slice's `status` and
its actual implementation state disagree before someone catches it, and
whether the readiness gate changes what gets built versus just when. That's
next engagement's work, not this one's — but naming the metrics now, before
a single pilot has run, is itself part of the honesty this story is built
on.

The claims here are demonstrated by shipped tooling, dated where they can
only be argued from a data point, and conceded where they're still just an
argument. That's not a modest launch — it's the actual differentiator. Every
other pitch in this space asks you to trust a roadmap. This one hands you a
CLI, a clone command, and a conformance report with a date on it, and asks
you to read the citations yourself.
