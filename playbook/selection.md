# Pilot selection

This is for the engineering lead deciding whether, and where, to run a
4-week Slicewright pilot. Work through it in one sitting: an eligibility
checklist, a list of situations to wait on, and a router that tells you
which of the two entry paths your team starts from. At the end you'll know
whether to pilot at all and, if so, whether you're going bridge-first or
full-loop.

If event modeling is new to you, read
[narrative/03](../narrative/03-describe-the-business-process.md) first —
ten minutes, and it teaches the notation this playbook is written in
(events, commands, views, the four slice patterns); the playbook's own
[README](README.md) fills in the last two terms, persona and swimlane, in
one paragraph.

## Eligibility checklist

Check every box before you commit a team to this. All five, not most of
five — a pilot missing one of these produces weak evidence even if it
finishes.

- [ ] **A bounded business capability with real invariants.** One slice of
  the business — order placement, cancellation, a single approval flow —
  small enough to model in a handful of slices, with rules worth checking
  (a total that must reconcile, a state that can't be reached twice, a
  window that closes). If nothing in the capability would surprise you to
  see violated, event modeling's completeness checking has nothing to
  earn its keep on. See [scope sizing](#pilot-scope-sizing) below.
- [ ] **A git + GitHub workflow.** `em ci init` scaffolds
  `.github/workflows/em-ci.yml` and `em-conform.yml` — **GitHub Actions
  only.** If your team runs GitLab CI, Jenkins, or another CI system, the
  generated workflows won't run there; you can still use the `em` CLI
  directly (`em validate`, `em status`, `em coverage`) without the
  scaffolded CI, but the one-command setup this playbook assumes doesn't
  apply. Flag that as a manual-CI pilot up front, not something you
  discover in week 2.
- [ ] **Someone with authority to ratify.** A named person who can decide
  "this slice is correct, build it" and stand behind that decision —
  `em slice ratify --by "<name>"` records their name and the date directly
  in the slice doc. See [roles.md](roles.md) for what the ratifier role
  actually does.
- [ ] **A codebase — or greenfield scope — sized for 4 weeks.** Either an
  existing codebase where the chosen capability is small enough to model,
  gate, implement, and run one conform cycle against inside a month, or a
  greenfield capability scoped the same way. Don't pick "rewrite the whole
  system" as pilot scope.
- [ ] **Willingness to run an agentic implement step.** The build phase
  runs through an AI coding agent — either the bundled `em skill install`
  facilitation skill, or the agent-neutral contract path (`em contract`,
  `AGENTS.md` discovery, `em export --slice`, the MCP server). State the
  agent-neutral path's evidence at its honest scope before you commit to
  it: it has been exercised once, by one agent, without the bundled skill,
  within one vendor family. A cross-vendor run hasn't happened. If your
  team's coding agent is a different vendor than the one that path was
  proven on, treat that as untested, not assumed-fine.

## Anti-selection: wait on these

None of these are permanent disqualifications — they're reasons to wait,
or to shrink scope, not reasons to abandon the idea.

- **No domain expert available for the pilot's duration.** Facilitation
  surfaces the model by asking the domain expert what's actually true;
  without one, the modeling sessions either stall or the agent starts
  inventing domain facts nobody validated. Wait until you have one.
- **Pure CRUD with no invariants worth checking.** If every operation is
  "store what the client sent, return it later," there's nothing for
  `em validate`'s gates or `em conform`'s drift detection to check beyond
  structure. Pick a capability with real rules instead, or don't pilot
  yet.
- **Can't commit the weekly cadence.** Ratification turnaround and the
  conform loop both depend on someone actually showing up on schedule —
  `em-conform.yml` runs weekly by default, and a pilot that skips weeks
  produces a thin, unrepresentative record rather than useful evidence.
  If nobody can commit roughly consistent weekly attention for 4 weeks,
  wait.
- **Windows-only tooling, no POSIX layer.** `em-sdd-bridge --symlink`
  creates a real filesystem symlink, which is POSIX behavior. Narrative
  Beat 8 notes the bridge has a non-symlink emission path for workflows
  where a filesystem symlink "doesn't fit" — but no recon run behind this
  playbook has exercised that path on Windows. If your team's dev
  environment is Windows without WSL or another POSIX layer, treat
  bridge-first as untested for you specifically, and either validate the
  non-symlink path yourself before committing, or route to full-loop
  (which doesn't depend on the bridge at all — see the router below).

## The entry-path router

Two entry paths, and the fork is about what your team already runs, not
about ambition. Pick a row:

| Your team today | Entry path |
|---|---|
| Already runs [spec-kit](https://github.com/github/spec-kit) SDD (spec → plan → tasks → implement) | **Bridge-first** — point `em-sdd-bridge` at one ratified slice; the smallest credible pilot. |
| Runs a different SDD tool (Kiro or similar) | **Full-loop.** The bridge's live validation covers exactly one SDD tool, spec-kit, for two real slices — a different tool is a documented gap, not a tested path. Don't be the first live test of an untried bridge integration during a pilot you're trying to get clean data from. |
| No SDD tool in place | **Full-loop** — model, gate, implement, conform, ratify, end to end. There's nothing else to install underneath it. |
| Unsure, or SDD adoption is partial/inconsistent across the team | **Full-loop, on a small scope.** Ambiguity is itself a reason to run the whole loop rather than bolt onto a workflow that isn't solid yet — you'll learn more from a small full-loop pilot than from a bridge pointed at an SDD process half your team doesn't actually follow. |

Both paths use the same slice doc as the source of truth and the same
readiness gate (`em validate --slice-ready`) before anything downstream
runs. The 4-week arc in [week-by-week.md](week-by-week.md) forks at the
point the paths actually diverge — everything before that point is
identical.

## Pilot scope sizing

"One capability, 5–10 slices" is the target size. Calibrate against
Meridian Goods, the reference implementation this playbook cites
throughout: **8 slices, 4 patterns, no exceptions** — one bounded capability
(order placement through cancellation), covering all four slice patterns
(State Change, State View, Automation, Translation) because that
capability naturally needed a state change, a couple of read-side views, a
reactive automation, and a boundary crossing to a payment provider. Your
capability doesn't have to hit all four patterns — a simpler capability
might be State Change and State View only — but if you're modeling more
than 10–12 slices for a first pilot, you've picked a scope too large to
gate, implement, and conform-check inside 4 weeks. Cut it down to the
smallest capability that still has a real invariant in it, not the
smallest feature.

Two sizing failure modes to watch for, both real risks, neither
hypothetical:

- **Too small**: a capability with one slice and no meaningful invariant
  gives you nothing for the conform loop or the readiness gate to
  demonstrate — you'll finish the pilot having proven the CLI runs, not
  that the loop catches anything.
- **Too large**: a capability that needs 20+ slices to model completely
  will not clear ratification, implementation, and one full conform cycle
  in 4 weeks. Narrow the capability boundary, don't stretch the timeline —
  the arc length is fixed by design.

## Next

- [roles.md](roles.md) — who does what once you've picked a capability and
  an entry path.
- [week-by-week.md](week-by-week.md) — the 4-week arc itself, forked at
  bridge-first vs full-loop.
- [metrics.md](metrics.md) — what to record from the pilot you're about to
  run.
- [governance.md](governance.md) — how the artifacts this pilot produces
  map onto an existing audit process.
