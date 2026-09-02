# Roles

This is for the four people (or fewer — see below) who'll actually run the
pilot once your team has cleared [selection.md](selection.md). It names
four roles, what each one is on the hook for, roughly how much time each
takes per week, and the exact `em` surfaces each one touches. By the end
you'll know who on your team should hold which role before week 1 starts.

The two loops these roles operate are described in
[narrative/06-two-loops.md](../narrative/06-two-loops.md): a demand loop
(facilitator proposes, ratifier decides) running on business time, and a
supply loop (implementing agent builds, conform checks) running on
technology time. The roles below map directly onto that split — facilitator
and ratifier run the demand loop, implementing agent and conform-cadence
owner run the supply loop.

## Can one person hold more than one role?

Yes, on a small team — nothing here requires four distinct people. The one
combination to avoid for the pilot's evidentiary value: **the ratifier and
the implementing agent's operator should not be the same person.** The
whole point of the ratification gate (Beat 6, Beat 8) is that someone other
than the person building the thing decides it's correct to build. Collapse
that distinction and you've collapsed exactly the separation of duties the
pilot exists to demonstrate — your conform findings and ratification
turnaround numbers (see metrics.md) will still be collectable, but they
won't mean what they'd mean with the roles actually separated. If your team
is too small to keep them apart, say so explicitly in your pilot notes
rather than letting it go unstated.

## Facilitator

**Mission.** Runs the modeling sessions that turn a stated need into a
model delta — new slices, changed slices, removed slices — using the
Socratic discover/model conversation, not code-first authorship. Keeps the
model rendering live so the rest of the team can see and challenge it as it
forms. No prior event-modeling experience required: the installed skill
bundle drives the session's questions; the facilitator hosts the
conversation and holds the pen. Read
[narrative/03](../narrative/03-describe-the-business-process.md) for the
notation (events, commands, views, the four patterns) and the playbook
[README](README.md)'s persona/swimlane paragraph before the first session.

**Responsibilities**

- [ ] Run the discover phase to draw out personas, commands, and events
  from whoever holds the domain knowledge — never invents a domain fact
  nobody supplied.
- [ ] Run the model phase to group the draft into contexts and classify
  each unit into one of the four patterns (State Change, State View,
  Automation, Translation).
- [ ] Add, change, or remove slices as new needs arrive during the pilot,
  each one a deliberate model delta, not a silent edit.
- [ ] Keep a live-rendered view running during sessions so the model is
  something the room watches, not a private file.
- [ ] Hand each drafted slice to the ratifier once it's complete enough to
  review — the facilitator proposes, they don't self-ratify.

**Weekly time commitment**: roughly half a day to a day per active
modeling week (session prep, the session itself, drafting slice docs
afterward) — heavier in week 1, lighter once the base model is ratified.
**This is a designed-in-advance estimate, not a measured one — no pilot
has run yet.**

**em surfaces**

- `/event-modeling discover` and `/event-modeling model` — the facilitation
  skill's session phases (from `em skill install`'s bundle).
- `/event-modeling slice` — the phase used to add a new slice for a change
  request, mid-pilot.
- `em slice new "<Slice Name>" --pattern <pattern> --swimlane "<swimlane>"
  --wire <model>.em` — scaffolds the slice doc's frontmatter and heading
  and wires its `note` binding into the model; the judgment sections
  (Intent, invariants, scenarios) are still written by hand — see
  [week-by-week.md](week-by-week.md) for the full sequence.
- `em watch <model> -o <svg> --serve --port <port>` — live, pan/zoomable
  rendering with a Review-mode toggle and live-reload, for the session
  itself.
- `em slice index` — regenerates the model's Slices table (a summary
  index of existing slices, not a scaffolding command).

## Ratifier

**Mission.** Decides whether a drafted slice is correct enough to build,
and whether a conform finding means the model bent or the code did.
Ratification is a named decision with a date attached, not a status a
tool flips on its own.

**Responsibilities**

- [ ] Review each slice the facilitator hands over and ratify it, or send
  it back with what's missing.
- [ ] Rule on every conform finding the conform-cadence owner brings
  forward — does the model bend (the model or doc was wrong; fix it) or
  does the code bend (the behavior was wrong; fix the implementation)?
  Each ruling lands as one of the concrete dispositions described in
  [week-by-week.md](week-by-week.md)'s Week 4 — same cycle, not left open.
- [ ] Stamp superseded conform reports once their findings are ratified,
  so a later reader doesn't mistake a ruled report's file:line citations
  for the model's current state.
- [ ] Advance the conformance marker at the close of each cycle — this is
  a human-only step; scheduled conform runs write reports but never move
  the marker themselves.
- [ ] Serve as (or set up) the `CODEOWNERS` reviewer on `slices/**` so
  ratification actually routes through you before it can merge.

**Weekly time commitment**: a few hours per week — spikier in review
weeks (ratifying a batch of drafted slices) than in conform weeks (ruling
on a handful of findings). **Designed-in-advance estimate, not measured.**

**em surfaces**

- `em slice ratify <model> <key> --by "<name>" [--on YYYY-MM-DD]` —
  flips `status: ready-to-implement` and records `ratifiedBy`/`ratifiedOn`;
  refuses to silently overwrite a different identity already recorded.
- `em slice reratify <model> <key>` — for a change cycle on a slice that's
  already `implemented`: bumps `version`, flips status back to
  `ready-to-implement`, clears the stale ratifier fields.
- A documented `CODEOWNERS` entry on `slices/**` (a GitHub convention, not
  an `em` command) routing the ratification flip through your review
  before merge.
- `em conform-supersede <model> <report> --as-of <rev> --findings <spec> [--on <date>]`
  — stamps a ruled report as superseded; additive, never rewrites the
  original findings.
- `em state set-conformance <rev> --report <path>` — advances the
  conformance marker at cycle close; human-invoked only.

## Implementing agent (+ its human operator)

**Mission.** Builds the ratified slice: an AI coding agent does the
implementation work, reading the slice doc as its contract, against a
human operator who reviews and merges what it produces. The agent proposes
code; it doesn't merge itself in.

**Responsibilities (the operator's)**

- [ ] Confirm the slice clears the readiness gate before pointing the
  agent at it — don't let implementation start on a slice that hasn't
  passed all four gates.
- [ ] Point the agent at the slice doc (bundled skill, or the agent-neutral
  contract path) and let it implement against the contract, not a
  paraphrase of it.
- [ ] Review the agent's output — tests included — before merging; the
  agent proposes, the operator is the human step that actually ships it.
- [ ] Run coverage in strict mode before merge so every cited invariant has
  a test behind it, not only a slice doc that claims one.
- [ ] Record the merge against the slice once the PR lands, so the slice
  doc's own history shows what shipped it.
- [ ] If your team is bridge-first: run the bridge against the ratified
  slice before starting implementation, so `plan`/`tasks`/`implement` run
  against the slice doc itself.

**Weekly time commitment**: the largest and most variable of the four —
plan for most of a normal engineering week during active implementation
weeks, since this is where the actual code gets written. **Designed-in-
advance estimate, not measured — expect this one to vary most against
reality.**

**em surfaces**

- `em validate <model> --slice-ready <key>` — the 4-gate readiness check
  (`docBound`, `frontmatterUsable`, `statusReady`, `noUncheckedOpenQuestions`);
  exit 0 only when all four pass.
- `em contract` — prints the implementation contract the agent works from.
- The MCP server (`em-mcp`, 13 read-only tools, byte-identical to their CLI
  equivalents): `validate`, `slice_ready`, `list_markers`, `export_model`,
  `export_slice`, `coverage`, `contract`, `status`, `diff`, `glossary`,
  `changelog`, `conform_scope`, `freshness`.
- `em coverage <model> --tests <dir> --strict` — fails unless every
  `INV-*` invariant is cited by a test.
- `em slice mark-implemented <model> <key> <pr-url>` — idempotent on the
  same PR URL; refuses to silently overwrite a different one.
- Bridge-first only: `npx -y em-sdd-bridge@0.4.1 <slice-key> --symlink` —
  hands the ratified slice doc to spec-kit as its spec, either as a real
  symlink or, where that doesn't fit, a generated file with literal
  invariant back-references.

Note on the agent-neutral path: an agent can work from `AGENTS.md`
discovery, `em export --slice`, and the MCP server instead of the bundled
skill. That path has been exercised once, by one agent, without the
bundled skill, within one vendor family — real evidence, but one data
point. State it to your operator at that size; don't assume it transfers
to a different agent vendor untested.

## Conform-cadence owner

**Mission.** Keeps the conform loop actually running on schedule and
keeps the model's freshness and status legible, so nobody discovers
three months of unratified drift at once. Triages what each conform run
finds and hands it to the ratifier to rule on — the owner doesn't rule on
findings themselves.

**Responsibilities**

- [ ] Confirm `em-conform.yml`'s weekly schedule is actually running (or
  triggered on demand) — a scheduled workflow nobody checks on is not a
  cadence.
- [ ] Check `em freshness <model>` regularly to see how many commits and
  slice-PRs the model is behind HEAD, and flag it before it grows large.
- [ ] Watch `em status <model> --tests <dir>` for the rollup line — slices
  implemented, invariants covered, open issues, conformance state — as the
  single source of truth for "is this healthy right now."
- [ ] Read each new `conformance/<date>-report.md` as it lands, sort
  findings from noise, and bring the real ones to the ratifier promptly
  rather than letting reports pile up unread.
- [ ] Confirm the conform workflow stays advisory — it opens a report (or
  an issue), it never fails the build and never rewrites the model itself.

**Weekly time commitment**: light in most weeks — an hour or so to check
freshness/status and skim a new report — heavier the week a report surfaces
real findings that need write-ups for the ratifier. **Designed-in-advance
estimate, not measured.**

**em surfaces**

- `em-conform.yml` — the scaffolded weekly cron + manual-dispatch workflow
  (`em ci init` generates it alongside `em-ci.yml`); runs an advisory
  conform sweep that never fails the build.
- `em freshness <model>` (+ `--json`) — "N commits and M slice-PRs behind
  HEAD."
- `em status <model> --tests <dir> [--json|--md|--badge]` — the rollup
  line, plus a markdown table or SVG badge for a README.
- `conformance/<date>-report.md` — the report itself, read directly; the
  owner triages it, the ratifier rules on it.

## Next

- [selection.md](selection.md) — eligibility and the entry-path router,
  before you assign these roles to actual people.
- [week-by-week.md](week-by-week.md) — where each role's responsibilities
  land on the 4-week calendar.
- [metrics.md](metrics.md) — what each role's activity contributes to the
  four Beat-9 metrics.
- [governance.md](governance.md) — how the ratifier's and conform-cadence
  owner's records map onto an existing audit process.
