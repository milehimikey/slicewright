# The adoption playbook

Everything a team that has never seen `em` needs to start a 4-week
Slicewright pilot from this directory alone. The [narrative](../narrative/)
makes the case and states its evidence; this playbook is the path from
"interested" to "running."

One thing to know before reading further, stated the way this project
states everything: **no pilot has run yet.** This playbook was designed in
advance — the roles, the arc, the metrics were all written before the first
team tried them, and every time estimate in it says so. What is *not*
designed-in-advance: every command in these documents was verified against
the published `@milehimikey/em@1.9.1` and `em-sdd-bridge@0.4.1` packages
before it was written down. The procedures are untested as a whole; the
commands are not.

## Before day 1: the notation

This playbook uses event modeling's vocabulary throughout.
**[narrative/03](../narrative/03-describe-the-business-process.md)** teaches
most of it in about ten minutes: the timeline, the three element kinds
(events, commands, views), the four slice patterns (State Change, State
View, Automation, Translation), and the two structural rules. Two terms it
shows without naming, defined here: a **persona** is who is acting —
Customer, Staff — and each persona gets its own horizontal row across the
top of the diagram; a **swimlane** is any of those horizontal bands
(persona rows above, system-context rows below), which is why a slice doc
records its home as `"Persona → Context"`. Everyone on the pilot reads it before day 1;
the facilitator and ratifier also read
[narrative/06](../narrative/06-two-loops.md) for the two loops their roles
operate. Nobody needs prior event-modeling experience beyond that — the
skill bundle installed on day 1 runs the modeling sessions Socratically,
asking the questions; the facilitator's job is to host that conversation,
not to already be an expert in it.

## Read in this order

1. **[selection.md](selection.md)** — should your team pilot at all, on
   what scope, and via which entry path (bridge-first, for teams already
   running spec-kit SDD, or full-loop, for teams without an SDD tool).
2. **[roles.md](roles.md)** — the four roles that run the pilot:
   facilitator, ratifier, implementing agent (+ operator), conform-cadence
   owner. Assign them to people before week 1.
3. **[week-by-week.md](week-by-week.md)** — the 4-week arc itself: day-1
   prerequisites, per-week goals, exact commands, and checkable exit
   criteria. This is the document the team executes.
4. **[metrics.md](metrics.md)** — the four measurements the pilot commits
   to collecting, named publicly before any pilot ran
   ([narrative/09](../narrative/09-early-credible-tryable.md)), with
   hand-collection procedures.
5. **[governance.md](governance.md)** — for whoever owns audit or
   compliance sign-off: what to hand a review board before the pilot, and
   what the audit trail looks like while it runs.

Reading load: the engineering lead reads all five; each role-holder needs
their own section of roles.md plus week-by-week.md; the compliance owner
needs governance.md alone.

## What you'll have at the end of 4 weeks

One bounded capability modeled as slices; every implemented slice ratified
by a named person before it was built; test coverage citing every
invariant; one completed conform cycle with every finding ruled on; a
first data point for each of the four metrics; and a go/no-go decision
made on evidence rather than impressions.

If the pilot goes well — or if it fails in an instructive way — the
metrics you collected are the methodology's first real track record, and
we want to hear about them either way: open an issue on this repository.
