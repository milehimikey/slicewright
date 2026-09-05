# Metrics

This is for your conform-cadence owner and your engineering lead. By the end
of it you will have four hand-collectable procedures — what to record, from
where, on what cadence — for the four metrics this pilot commits to
tracking, plus a plain markdown template your team can copy into its own
notes and start filling in on day one.

## Why these four, and why they're hand-collected

These four metrics — conform-cycle cadence and finding counts,
ratification turnaround, status-vs-reality disagreement, and the
readiness gate's effect on what gets built — were named publicly in
[narrative/09](../narrative/09-early-credible-tryable.md)'s "What we're
asking for" section, **before any pilot had run**. That section says
plainly: naming the metrics in advance, rather than promising something
vaguer later, is itself part of the honesty the whole methodology is
built on. This document is that promise kept — the operational form of
the same four metrics, nothing added and nothing substituted.

No `em metrics` command exists in em 1.10.0. Everything below is a
procedure a person runs by hand, on a cadence, against artifacts `em`
already produces. A future em release may automate some of this
collection; until it does, collecting these four numbers by hand is part
of what a pilot signs up for — and a pilot that does so honestly
contributes this methodology's first real track record, not a second
data point stapled onto a claim that already outran its evidence.

## 1. Conform-cycle cadence + finding counts

**Source:** the `conformance/<date>-report.md` file each conform cycle
produces, and that report's own Summary matrix (a table breaking findings
down by surface — Structural / Spec / Internal — and by classification —
real drift / model gap / internal inconsistency / accepted divergence /
unpropagated delta / uncertainty).

**What to record, per cycle:**

- the report's date (its filename) and the target revision it checked
  against (the report's own "Target repo … @ `<rev>`" line)
- findings by classification, read straight off the Summary matrix — do
  not recount by hand, the report already tallies this
- false positives — not a field the tool computes; it's a human judgment
  the ratifier writes into the report's own **Ratification record**
  section when they rule on each finding. Meridian Goods' first cycle is
  the pattern to repeat: 4 findings, each with a dated ruling in the
  report itself, and 0 false positives — a count the
  [narrative's claims register](../narrative/09-early-credible-tryable.md)
  states from that record, not a number to expect from your own runs

**Cadence:** once per conform cycle. A pilot running `em ci init`'s
generated `em-conform.yml` gets a weekly advisory cadence by default (see
[week-by-week.md](week-by-week.md)); record this row every time a report
lands, scheduled or manually dispatched.

## 2. Ratification turnaround

**Source:** git history. `em slice ratify <model> <key> --by "<name>"`
writes `ratifiedBy:`/`ratifiedOn:` into the slice doc's own frontmatter,
which makes the ratification moment queryable as a commit, not just a
date field to trust at face value.

**What to record, per slice:** the time from the commit that created the
slice doc (or, for a change cycle, the commit where `em slice reratify`
last cleared `ratifiedBy:`) to the commit where `em slice ratify --by`
set it again.

**Command pattern** (verified against a scratch model and slice doc,
`em 1.10.0`, `git log` on the resulting repo):

```bash
# the commit that first added the slice doc
git log --follow --diff-filter=A --format='%h %ad' --date=short \
  -- slices/<slice-key>.md

# the commit where `ratifiedBy:` first appears in that doc
git log -S"ratifiedBy:" --follow --reverse --format='%h %ad' --date=short \
  -- slices/<slice-key>.md | head -1
```

The first command finds the doc's creation commit (`--diff-filter=A`
scopes to the add). The second uses git's pickaxe search (`-S`) to find
the first commit that introduced the literal string `ratifiedBy:` into
that file's history, and `--reverse` walks oldest-first so `head -1`
gives you the *first* ratification rather than the most recent one. The
gap between the two dates is the turnaround. In the scratch run that
built this pattern, a slice doc created 2026-08-01 and ratified
2026-08-15 gave the expected ~14-day gap end to end.

For a slice going through a **change cycle** — `em slice reratify` bumps
`version:`, flips `status` back to `ready-to-implement`, and clears the
prior `ratifiedBy:`/`ratifiedOn:` — replace the first command with the
commit that ran `reratify` (findable the same way, by pickaxing on the
new `version:` string, or by commit message if your team names them
consistently), and use the same `-S"ratifiedBy:"` search restricted
to commits after it (`git log ... <reratify-commit>..HEAD`) to find the
next ratification.

**Cadence:** compute per slice, at ratification time or in your weekly
roll-up — whichever your conform-cadence owner prefers; either produces
the same numbers.

## 3. Status-vs-reality disagreement

**Source:** `em status <model> --json`'s `driftSignal` breakdown
(`inSync` / `neverImplemented` / `unpropagatedDelta` /
`implementedWithoutLink`, plus `notApplicable` and `frontmatterInvalid`
for slices with no usable doc), sampled weekly.

**What to record:** any slice whose `driftSignal` bucket disagrees with
what your team knows to be true on the ground — most concretely, a slice
sitting in `unpropagatedDelta` (code moved, the doc didn't catch up) or
`implementedWithoutLink` (doc claims `implemented` with no
`implementedIn` anyone can point to) that a conform cycle or a person
catches before the tool's own weekly sample would have. Record: the
slice key, which bucket it was in, what the actual state turned out to
be, and how it was caught (conform cycle, PR review, someone noticing by
hand).

**Cadence:** weekly. `em status <model> --tests <dir> --json` is
deterministic and cheap to run; a fixed day (tie it to the same day as
your conform-cadence owner's other duties, see
[roles.md](roles.md)) keeps the sample comparable week to week.

## 4. Gate effect on what gets built

**Source:** no tool output — this one is qualitative by design, per the
pilot invitation's own framing ("whether the readiness gate changes what
gets built versus just when"). The record is a weekly log your
facilitator or conform-cadence owner keeps.

**What to record, per week:** how many slices were proposed, how many
were ratified, and how many were refused by the gate — a
`em validate <model> --slice-ready <key>` run that came back non-zero and
sent the slice back for rework rather than through to an implementer —
with one line on *why* each refusal happened (missing doc binding, an
unresolved Open Question, status not yet `ready-to-implement`, or a
substantive judgment call the ratifier made that isn't one of the gate's
own four checks).

**Cadence:** weekly, alongside the metric-3 sample.

## Collection cadence

| Metric | What to record | When | Who |
|---|---|---|---|
| Conform cadence + findings | date, target revision, findings by classification, false positives | every conform cycle | conform-cadence owner ([roles.md](roles.md)) |
| Ratification turnaround | creation/reratify commit → ratify commit, per slice | at ratification, or rolled up weekly | ratifier or conform-cadence owner |
| Status-vs-reality disagreement | slice, driftSignal bucket, actual state, how caught | weekly | conform-cadence owner |
| Gate effect on what gets built | slices proposed / ratified / refused-by-gate, one line per refusal | weekly | facilitator or conform-cadence owner |

## Recording template

Copy this into your pilot's own notes (a shared doc, a `metrics/` folder
in the repo — this playbook doesn't prescribe where) and fill it in as
you go.

```markdown
## Conform cycles

| Date | Target rev | Real drift | Model gap | Internal inconsistency | Uncertainty | False positives |
|---|---|---|---|---|---|---|
|      |             |            |           |                         |             |                  |

## Ratification turnaround

| Slice key | Created/reratified (commit, date) | Ratified (commit, date) | Turnaround |
|---|---|---|---|
|           |                                    |                          |            |

## Status-vs-reality disagreement (weekly sample)

| Week of | Slice key | driftSignal bucket | Actual state | Caught by |
|---|---|---|---|---|
|         |           |                     |               |           |

## Gate effect on what gets built (weekly)

| Week of | Proposed | Ratified | Refused by gate | Why (one line each refusal) |
|---|---|---|---|---|
|         |          |          |                  |                              |
```

These templates are a starting point, not a schema `em` enforces —
adjust column widths to your team's own vocabulary, but keep the four
sections separate, since they answer four different questions and mixing
them muddies all four.
