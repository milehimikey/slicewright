# Week-by-week: the 4-week pilot arc

This is the execution plan for a team that has never seen `em` before. It
assumes you've already read [`selection.md`](selection.md) and picked an
entry path — **bridge-first** (you already run an SDD tool like spec-kit) or
**full-loop** (you don't). By the end of Week 4 you'll have one or more
slices modeled, ratified, implemented, and checked once against the running
code, plus a first data point for each of the four pilot metrics and a
go/no-go decision.

Every command below is quoted exactly as it runs against `@milehimikey/em@1.10.0`
and `em-sdd-bridge@0.4.1` — verified in scratch space against the published
packages, not transcribed from memory. Roles are named but not re-described
here; see [`roles.md`](roles.md) for responsibilities, time commitment, and
which `em` surfaces each role operates.

## Week 0 / Day 1 — Prerequisites

Do this once, before Week 1 starts.

**Install, pinned:**

```sh
npm install -g @milehimikey/em@1.10.0
```

Or run any command via `npx` without a global install — pin the version on
every invocation:

```sh
npx @milehimikey/em@1.10.0 <command>
```

Bridge-first teams: nothing to run for the bridge today — its first real
invocation is Week 2, once a ratified slice exists to point it at. Just
write the pinned form into your pilot notes now so nobody improvises an
unpinned one later:

```sh
npx -y em-sdd-bridge@0.4.1 <slice-key> --symlink
```

**Repo setup:**

- Full-loop, starting fresh: `em scaffold <name>` writes `<slug>/<slug>.em`,
  a starter `README.md`, and `.event-modeling.md` (the resumable session
  state file) in one step. Pass `--under <dir>` if this model will live
  alongside others in a shared parent directory.
- Bridge-first, or full-loop adding a model to an existing repo: `em init
  [file]` (default `model.em`) writes just the starter `.em` file — use this
  when you don't want the README/`.event-modeling.md` scaffold `em scaffold`
  also produces.

**Install the skill bundle:**

```sh
em skill install
```

This copies the 7-directory skill bundle into `.claude/skills/`
(`event-modeling` router + `-discover`/`-design`/`-implement`/`-conform`/
`-review` + `-shared`) and writes an agent-contract section into
`AGENTS.md`, pointing any agent — Claude Code or otherwise — at `em
contract` for the implementation contract. You always invoke phases
through the router — `/event-modeling <phase>`, with phase names
`discover`, `model`, `slice`, `implement`, `conform`, `review` — and the
router maps each to the right skill directory (the directory names don't
match the phase names one-to-one: `model` and `slice` both live in
`-design`). Bare `/event-modeling` with no phase resumes wherever the
state file says you left off. Teams not using Claude Code read
that same contract via `em contract` on the command line or the MCP
server's `contract` tool; the skill isn't a hard requirement, `em contract`
is the agent-neutral fallback.

**Route ratification through a designated reviewer:**

Create a GitHub team for your ratifier(s) first if one doesn't exist —
CODEOWNERS can only name handles that already resolve. Then add a
`CODEOWNERS` entry for the slice-doc directory, and turn on "Require
review from Code Owners" in your branch protection rule:

```text
slices/**  @your-org/ratifiers
```

`em slice ratify` mechanizes the frontmatter *edit* — it can't by itself stop
someone hand-editing the same YAML. CODEOWNERS is the mechanical half:
GitHub refuses to merge a PR touching `slices/**` without approval from
whoever the file names. List a team, not one person, so the pilot doesn't
stall the moment your ratifier is out. Route the *directory*, not the
command — there's no way to distinguish a ratification produced by `em
slice ratify` from a hand-edit; both are just a diff under `slices/**`, so
that's the path to gate.

**Scaffold CI:**

```sh
em ci init <model>.em --tests <dir>
```

`<dir>` is your test source root — `src/test/kotlin` for a Gradle Kotlin
project, `src/test/java` for Maven-layout Java, `test/` for many Node
projects. It's the same directory you'll pass to `em coverage` all pilot
long, so pick it once here.

This writes `.github/workflows/em-ci.yml` (PR gates: validate, slice-index
`--check`, coverage `--strict`, ledger, skill-check, glossary, plus a
push-triggered status-badge rebuild) and `em-conform.yml` (a weekly
cron + manual-dispatch, advisory-only conform sweep — see Week 4). Both
files are marker-delimited and idempotent; rerun with `--check` any time to
verify they still match the current preset without writing.

**Version pinning is automatic at `@milehimikey/em@1.10.0` and later.** `em
ci init` scaffolds every `npx @milehimikey/em@<command>` line in
`em-ci.yml`, and the `npm i -g @milehimikey/em@<version>` line in
`em-conform.yml`, already pinned to the exact version of `em` that generated
them — there's no hand-pinning step left to run. Verify it rather than
trust it, since it costs nothing:

```sh
em ci init <model>.em --tests <dir> --check
```

prints `ok — both workflow files match the current preset` when the pins
are current; it never writes, so it's safe to run any time, including as a
CI check of its own. (If your team is still on `em` 1.9.x: scaffolds from
that version are unpinned — a floating `npx @milehimikey/em <command>` and
a major-only `npm i -g @milehimikey/em@1` — so either pin every generated
`run:` line by hand before committing, or upgrade to 1.10.0 first and
re-run `em ci init` to get the self-pinned files for free.)

**Week 0 exit criteria:**

- [ ] `em --version` prints `1.10.0` (or `npx @milehimikey/em@1.10.0
      --version` resolves without error)
- [ ] a `.em` model file exists in the repo (`em scaffold` or `em init`)
- [ ] `.claude/skills/event-modeling*` exists and `AGENTS.md` has a managed
      agent-contract section
- [ ] `CODEOWNERS` names a ratifier team on `slices/**`, and branch
      protection requires their review on that path
- [ ] `.github/workflows/em-ci.yml` and `em-conform.yml` exist, and
      `em ci init <model>.em --tests <dir> --check` reports `ok` for both

## Week 1 — Model and ratify the first slice(s)

**Goal:** the demand loop's first lap — a slice goes from nothing to a
ratified, `ready-to-implement` doc. **Who:** facilitator runs the session;
ratifier signs off at the end (see [`roles.md`](roles.md)).

### Bridge-first

Scope this to the one slice you're piloting — the small ask, per the
project's own invitation, is "point the bridge at one slice." Run a short
discover/model session focused on just that slice (in Claude Code:
`/event-modeling discover` then `/event-modeling model`; agent-neutral:
work from `em contract`'s reference material by hand), until the slice and
its pattern (State Change, State View, Automation, or Translation) are
settled in the `.em` file. Then scaffold its doc:

```sh
em slice new "<Slice Name>" --pattern <pattern> --swimlane "<swimlane>" --wire <model>.em
```

Placeholders, exactly as the CLI accepts them: `--pattern` takes one of
`state-change | state-view | automation | translation` (the kebab-case
tokens for the four patterns); `--swimlane` takes the `"Persona → Context"
form, e.g. `"Customer → Ordering"`. `--wire` inserts the
`note "slices/<key>.md"` binding onto the slice's primary element in the
model automatically, instead of printing it to paste by hand.

The scaffold writes only the five required frontmatter keys
(`schemaVersion`, `pattern`, `swimlane`, `status`, `version`) and the
heading. Write the judgment sections by hand — Intent, Command/Event/View
tables, named invariants, Scenarios, Open Questions. Invariant IDs follow
`INV-<MNEMONIC>-<n>` where `<MNEMONIC>` is a short (2–4 letter) abbreviation
unique to this slice — `INV-PO-1` for a place-order slice, `INV-CO-2` for
cancel-order — and `<n>` counts up within the slice.

```sh
em validate <model>.em
```

expect `ok — no issues`. Then the ratifier signs off:

```sh
em slice ratify <model>.em <slice-key> --by "<ratifier name>"
```

This flips `status` to `ready-to-implement` and records `ratifiedBy:` /
`ratifiedOn:`. It's idempotent on the same name, and it refuses outright to
overwrite a different name already recorded — so a second person can't
quietly re-ratify over the first. Nothing bridges yet; that's Week 2.

### Full-loop

Run the full discover/model session for the first slice(s) of the process
you're piloting — typically the happy-path spine, one or two slices'
worth:

```sh
/event-modeling discover
/event-modeling model
```

(Agent-neutral teams: the same phases, worked by hand from `em contract`'s
reference material.) Render it for the room as you go:

```sh
em watch <model>.em -o <model>.svg --serve --port 5173
```

For each slice that comes out of the session:

```sh
em slice new "<Slice Name>" --pattern <pattern> --swimlane "<swimlane>" --wire <model>.em
```

then hand-write Intent, the field tables, named invariants, Scenarios, and
Open Questions. Validate and ratify exactly as in the bridge-first path
above:

```sh
em validate <model>.em
em slice ratify <model>.em <slice-key> --by "<ratifier name>"
```

### Week 1 exit criteria

- [ ] `em validate <model>.em` reports `ok — no issues`
- [ ] the pilot's first slice(s) have slice docs at `status:
      ready-to-implement`, with `ratifiedBy:` / `ratifiedOn:` recorded
- [ ] at least one real PR touching `slices/**` was actually blocked or
      approved by the CODEOWNERS rule from Week 0 — confirm the routing
      works before relying on it for the rest of the pilot

## Week 2 — Implement through the gate

**Goal:** 2–3 slices (or, for a bridge-first single-slice pilot, that one
slice) go from `ready-to-implement` to `implemented`, through the readiness
gate, with cited test coverage. **Who:** implementing agent builds; ratifier
is available if the agent surfaces a gap instead of guessing.

Before any implementation work starts, on either path, confirm the gate:

```sh
em validate <model>.em --slice-ready <slice-key> --json
```

`ready: true` and all four gates true (`docBound`, `frontmatterUsable`,
`statusReady`, `noUncheckedOpenQuestions`) means go. Anything else, stop —
see [When things go wrong](#when-things-go-wrong) below.

### Bridge-first

```sh
npx -y em-sdd-bridge@0.4.1 <slice-key> --symlink
```

This allocates a spec-kit feature branch and `specs/NNN-slug/` directory
exactly as your existing spec-kit flow would, but instead of rendering a
new `spec.md`, it replaces it with a relative symlink straight to the
ratified slice doc. Nothing is generated or duplicated — the slice doc *is*
the spec your spec-kit phases read. The command gates on `em validate
<model>.em --slice-ready <key>` itself before allocating anything, and
prints a `**Traceability**:` line on success — paste it into the PR
description, since there's no rendered header to carry it.

Run your project's existing spec-kit phases (`plan`, `tasks`, `implement`)
against that symlinked spec as usual. One vocabulary note: prompts written
against spec-template section names (acceptance scenarios, `FR-00N`) now
read the slice doc's own sections instead (`## Scenarios`, `## Invariants`)
— a short preamble in your phase-prompt overrides covers the mapping.

Write tests that cite each invariant's `INV-<MNEMONIC>-<n>` id. "Cite"
means the literal ID string appears in a test source file under `--tests` —
the working convention is to put it in the test's own name, e.g. a Kotlin
backtick method name like
`` fun `INV-PO-1 - given no payment, when Place Order, then rejected`() ``
or a JS/JUnit display name — so the citation survives in test reports too.
Then:

```sh
em coverage <model>.em --tests <dir> --strict
```

exits non-zero if any declared invariant has zero test citations. At merge:

```sh
npx -p em-sdd-bridge@0.4.1 em-sdd-mark-implemented <slice-key> <pr-url>
```

(a thin wrapper that shells out to `em slice mark-implemented` itself —
one writer implementation, not two). Idempotent on the same PR URL;
refuses a second, different URL for an already-marked slice.

### Full-loop

Point the implementing agent at the contract before it starts:

```sh
em contract
```

(or the MCP server's `contract` tool — same content, either transport; see
[em's docs/mcp.md](https://github.com/milehimikey/em/blob/main/docs/mcp.md)
for the parity invariant, 15 read-only tools total). Implement the
slice, writing tests that cite `INV-<MNEMONIC>-<n>` ids, then:

```sh
em coverage <model>.em --tests <dir> --strict
```

At merge:

```sh
em slice mark-implemented <model>.em <slice-key> <pr-url>
```

Repeat for slice 2 and 3. This is the same command in both cases — the only
difference between the paths is how the spec gets to the implementing
agent (bridge symlink vs. the model/skill directly).

### Week 2 exit criteria

- [ ] `em validate <model>.em --slice-ready <slice-key> --json` returned
      `ready: true` before implementation started, for every slice
      implemented this week
- [ ] `em coverage <model>.em --tests <dir> --strict` exits 0
- [ ] `em slice mark-implemented` (or the bridge's wrapper) has run for
      every merged slice, and each doc's `implementedIn:` points at the
      real merged PR
- [ ] `em status <model>.em --tests <dir>` shows this week's slices as
      implemented

## Week 3 — Scale up, and run one deliberate change cycle

**Goal:** repeat the model → ratify → implement loop for a couple more
slices, and put one already-implemented slice through a real change —
proving the demand loop's revision path, not just its first pass. Both
entry paths converge here: from this point on there's one procedure.

Continue Week 1–2's loop for 2–3 more slices (bridge-first pilots scoped to
one slice can treat this as optional — the point of a one-slice pilot is
the bridge mechanics, not slice count). Same commands, not repeated here.

**The change cycle.** Pick a real change request against a slice already
`implemented`. Before the facilitator and ratifier decide the change, see
what it would touch: `em query downstream <model>.em --of "<element name>"`
walks the transitive closure of legal edges from the element the change
starts at — consumers, downstream views, reactions — one command instead of
tracing the diagram by eye. Then decide the change together, and hand-write
a `## Delta` section in the slice doc recording what changed and why — this
is a judgment section `em` never writes for you. Then:

```sh
em slice reratify <model>.em <slice-key>
```

This bumps `version:`, flips `status` back to `ready-to-implement`, and
clears any stale `ratifiedBy:` / `ratifiedOn:` (they described the *prior*
version's sign-off). It only applies to a doc currently `implemented`.
Ratify again:

```sh
em slice ratify <model>.em <slice-key> --by "<ratifier name>"
```

Gate again, exactly as in Week 2:

```sh
em validate <model>.em --slice-ready <slice-key> --json
```

Implement the change, then re-run `em slice mark-implemented` with the new
PR's URL — same command, and the same conflict refusal applies if a
different URL is ever passed for the same slice without an intervening
reratify. Close the loop with the structural and business-readable views of
what changed:

```sh
em diff <model>.em --from <old-rev> --to <new-rev>
em changelog <model>.em
```

### Week 3 exit criteria

- [ ] 2–3 additional slices implemented and marked this week (or: skipped,
      by design, for a scoped single-slice bridge-first pilot)
- [ ] one slice has gone through a full reratify → re-ratify → re-gate →
      re-implement → re-mark cycle
- [ ] that slice's doc has a `## Delta` section and `version:` bumped
- [ ] `em changelog <model>.em` shows the change as a dated, readable entry

## Week 4 — First conform cycle, metrics, go/no-go

**Goal:** run the pilot's first check of code against model, collect a
first data point for each pilot metric, and decide whether to continue.
**Who:** conform-cadence owner runs and reports; ratifier rules on every
finding, stamps the superseded report, and advances the conformance
marker; the whole team holds the retro.

**Run conform.** Either let Week 0's scheduled `em-conform.yml` fire (weekly
cron, or trigger it by hand via `workflow_dispatch`), or run the phase
directly:

```sh
/event-modeling conform
```

State this plainly to the team before the first report lands: **conform is
advisory.** Nothing it produces fails a build or blocks a merge on its own
— a false accusation of drift would cost the loop more trust than real
drift justifies, so the report only ever proposes.

**Rule on the report.** It lands at `conformance/<date>-report.md`. Every
finding falls into one of three buckets:

- the code is right and the model is stale — fix the model directly, no
  red note needed, the disagreement is resolved
- there's a real open decision — apply the report's proposed `issue
  "conformance: …"` red note (an `issue` marker in the `.em` file, rendered
  as a red folded-corner flag on the diagram); it stays visible until
  someone rules on it
- the doc is stale prose — fix the wording; nothing structural to record

Record what you decided, and when, in the state file's decisions log —
that's what `em changelog` later weaves back into the business-readable
ledger. Once every finding in the report has a ruling:

```sh
em conform-supersede <model>.em conformance/<date>-report.md --as-of <rev> --findings <spec>
```

This stamps the report with a "superseded as of `<rev>`" banner — additive
only, it never rewrites the report body — so anyone who opens it later
knows its file:line citations describe an ancestor of the current model.
Then, and only then, advance the marker:

```sh
em state set-conformance <rev> --report conformance/<date>-report.md
```

This step is **human-only, by design.** The scheduled `em-conform.yml` job
writes reports; it never advances this marker itself — that's the one edit
in this whole arc that stays deliberately outside automation, because it's
the record of a human having actually read and ruled on the findings.

**Collect the metrics.** Walk each of the four Beat-9 metrics' hand-
collectable procedures now — see [`metrics.md`](metrics.md) for what to
record, from where, and when. Do this even with only one week's data; the
point of Week 4 is a first recorded data point per metric, not a trend.

**Go/no-go retro.** Bring the engineering lead and all four role-holders
together. Pull up the shared state-of-the-system view as the anchor for the
conversation:

```sh
em status <model>.em --tests <dir> --md
```

Decide, and record the decision: continue past the pilot, adjust and
re-run a week, or stop.

### Week 4 exit criteria

- [ ] at least one conform report exists at `conformance/<date>-report.md`
- [ ] every finding in that report has a recorded ruling (fix the model /
      apply a red note / fix the wording) and a dated decisions-log entry
- [ ] `em conform-supersede` has run against that report
- [ ] `em state set-conformance` has run — by a person, confirmed not by
      the scheduled workflow
- [ ] every metric in `metrics.md` has at least one recorded data point
- [ ] the go/no-go retro happened and its decision is written down

## When things go wrong

### The readiness gate refuses a slice

```sh
em validate <model>.em --slice-ready <slice-key> --json
```

names exactly which of the four gates failed — `docBound` (the doc doesn't
resolve), `frontmatterUsable` (the frontmatter doesn't parse),
`statusReady` (status isn't `ready-to-implement` yet), or
`noUncheckedOpenQuestions` (at least one Open Question is still
unresolved) — each with a diagnostic message and line number. Read the
diagnostics; they are the spec of what's missing, not a generic failure to
work around. A slice that fails `statusReady` needs a ratifier's sign-off,
not a hand-edit of the status field — route it back through `em slice
ratify`. On the bridge-first path, `em-sdd-bridge` delegates to this exact
gate as its own precondition and relays `em`'s diagnostic text verbatim, so
the same reading applies there too.

### A conform finding feels wrong

If the evidence is genuinely ambiguous — the code path is unclear, a test
doesn't settle it either way, you ran out of time to chase it down — that's
an **uncertainty**, and it gets reported as one, never as drift and never
silently dropped. The rule that governs conform findings generally is the
same one: **uncertainty is never drift.** Don't suppress a finding because
it feels wrong on first read, and don't promote an uncertainty to a ruled
finding just to close it out faster — rule on what the evidence actually
shows, and mark what it doesn't as uncertain.

### The status table looks stale

A README's generated Slices table can drift from the model between
sessions if someone regenerates it inconsistently, or not at all. Week 0's
CI preset catches this on every PR:

```sh
em slice index <model>.em --check
```

exits non-zero, without writing, if the table doesn't match `em export`'s
current facts — so a stale table fails the PR gate instead of sitting
unnoticed until someone stumbles on it. Regenerate it locally with `em
slice index <model>.em` (no `--check`) before pushing.
