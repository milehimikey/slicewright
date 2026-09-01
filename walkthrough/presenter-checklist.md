# Presenter checklist

Run through this in full at least once, in a fresh clone, before any live
or recorded run of `script.md` or `teaser-5min.md`. Every item below was
verified against a real dry-run against the pinned versions; none of it
is speculative.

## Versions — pin and show them

- [ ] `em --version` → `1.8.0`. Show this on screen (a terminal line, or a
      slide corner) at the top of the talk.
- [ ] The bridge has **no** `--version` flag — it errors on a missing
      spec-kit scaffold instead of printing a version. Cite
      `npm view em-sdd-bridge@0.4.0 version` beforehand, or treat the
      pinned invocation itself (`npx -y em-sdd-bridge@0.4.0 …`) as the pin.
      Don't try `em-sdd-bridge --version` live — it doesn't exist.

## Repository state — the two places this script steps off `main`

Real `main` HEAD has every base and change-beat slice already
`implemented` — there is no single point in true history where a slice is
still `ready-to-implement` *and* real Kotlin code exists to satisfy the
bridge's own precondition. The script uses two different, disclosed
substitutions instead of one tag for the whole gate→bridge arc — get both
right before going live:

- [ ] **Beats 3–6** (render, slice doc, validate/break-fix, readiness
      gate): `git checkout walkthrough/01-model-ratified` — this is real,
      unedited history (the commit where the doc was ratified, before any
      code existed). The gate's FAIL half needs one further **local,
      uncommitted** edit on top (`place-order.md`: `status: draft`,
      uncheck one Open Question) — revert it before Beat 7's transition.
- [ ] **Beats 7–8** (bridge `--symlink`, refusal, `mark-implemented`):
      switch to a fresh clone of real `main` HEAD (code must already
      exist — the bridge's events-first precondition needs the event's
      Kotlin class already in the source tree, which `walkthrough/01` does
      not have). Locally, **uncommitted**, revert `place-order.md`'s
      `status` from `implemented` back to `ready-to-implement`. Beat 8's
      `mark-implemented …/pull/7` call restores this file to *exactly*
      real `main` HEAD (same URL as the real merge) — verified byte-
      identical after the run. For Beat 7b, also locally set
      `cancel-order.md`'s `status` to `draft`; restore it with
      `git checkout -- slices/cancel-order.md` once 7b is done (it isn't
      self-healing the way place-order is).
- [ ] Beats 9 onward run against real, unmodified `main` HEAD.
- [ ] Beat 10 starts from `git checkout walkthrough/03-all-base-implemented`
      (pre-change-beat); Beat 11's diff uses the real commit pair
      `238d324` → `7230315` either side of the real change-beat PR.
      Beat 14 starts from `walkthrough/06-conformance-report` (the report,
      pre-ratification); Beat 15's real-record comparison uses
      `walkthrough/07-ratified`.
- [ ] **Correction to the earlier working assumption**: "gate-pass and
      bridge beats both replay from `walkthrough/01-model-ratified`" is
      only half true — verified live that tag has no `.specify/` and no
      compiled event classes, so the bridge cannot run there at all (its
      events-first precondition fails outright). Only the gate-pass half
      (Beat 6) replays cleanly from that tag; the bridge beats (7, 7b, 8)
      need the `main`-HEAD-plus-local-revert substitution above. Don't
      re-introduce the single-tag version of this instruction.

## Spec-kit scaffold — verify BEFORE the bridge beat

- [ ] Confirm `.specify/em-sdd.json` (should read
      `{"contractSource": "none"}`) is present in whichever clone runs
      Beat 7, **before** going live. A missing scaffold produces a raw,
      ugly stack trace on stage, not a clean refusal — this is a known
      product rough edge (the bridge's scaffold precondition isn't
      graceful yet), not something to trigger live to "prove a point."
- [ ] Do not attempt a from-zero `specify init` live — real prerequisites
      (script-filename version coupling, a compiler dependency, the
      events-first gate needing a real code stub already in source) make
      that a multi-point failure risk. Scaffold prep is a pre-show task,
      not a beat.

## Ports and processes

- [ ] Pick a free port for `em watch --serve` before the show (`4173` used
      throughout this script/dry-run); confirm nothing else is bound to
      it. `pkill -f "em watch"` cleanly stops the server between runs —
      verify the served page 404s/refuses afterward before moving on.
- [ ] `em watch --serve` rewrites `meridian-goods.svg`'s highlight
      coloring on every render (a recency/highlight heuristic tied to
      system date, not a bug worth narrating). Expect a diff after every
      live render; `git checkout -- meridian-goods.svg` before committing
      or handing control back, or simply don't `git add` it during the
      show.

## Tags — fetched and verified

- [ ] `git fetch origin --tags` in the demo clone; confirm all seven
      `walkthrough/01`…`07` tags resolve and match this table (verified
      once already — re-verify after any force-push or history rewrite):

  | Tag | Commit | PR |
  |---|---|---|
  | `walkthrough/01-model-ratified` | `c858bbe` | #4 |
  | `walkthrough/02-first-slice-implemented` | `9a70548` | #7 |
  | `walkthrough/03-all-base-implemented` | `238d324` | #13 |
  | `walkthrough/04-change-beat` | `6635462` | #16 |
  | `walkthrough/05-drift-shipped` | `8f12ed8` | #17 |
  | `walkthrough/06-conformance-report` | `07c22bc` | #18 |
  | `walkthrough/07-ratified` | `35a53b5` | #20 |

## The seeded-divergence checkpoint (Beats 15–16)

- [ ] Real `main` HEAD's model file has **never**, at any point in real
      history, carried an in-model `issue "..."` or `divergence "..."`
      marker — the real ratification happened as prose (the conformance
      report) plus a doc-frontmatter version bump, not a DSL annotation.
      `--list-issues`/`--list-divergences --json` on real `main` return
      empty `markers: []` — this is expected, not broken; say so on
      stage rather than letting an empty result look like a demo failure.
- [ ] Keep a pre-seeded scratch copy on hand (or seed it live, ~10
      seconds) with the exact marker text this script uses, mirroring the
      real Beat 14 finding word-for-word — confirmed working syntax:
      `event Order Cancelled @Ordering issue "conformance: code allows
      cancel within a 24h grace window after Payment Captured; doc and
      domain-decisions.md both still say capture is an absolute bar"`,
      later replaced in place with `divergence "grace window ratified
      2026-08-23: …"`. Do not present this as real history — the script
      discloses it as a staged illustration; keep that framing intact.

## Local-branch hygiene

- [ ] Prune any local-only branches in both the `meridian-goods` and
      `em-sdd-bridge`-adjacent clones before any screen-share — verify
      with `git branch -vv` that nothing stale or day-job-named is
      sitting on the machine's branch list where a screen-share could
      catch it. The pushed `origin` remotes are the source of truth and
      are already clean; local cruft is a self-inflicted risk only.
- [ ] Re-verify repo-local git identity in every fresh clone before the
      first commit of the session: `git config user.name milehimikey &&
      git config user.email milehimikey@gmail.com`, confirmed after with
      `git log -1 --format='%ae'` → `milehimikey@gmail.com`. This machine's
      global git config carries a different identity — never assume a
      fresh clone inherited the right one.

## Known Q&A landmines — route around, don't narrate as fixed

- [ ] **UI-field tracing is unshipped.** `em validate`'s fields-completeness
      check traces `view`↔`event` and `event`↔`command` fields, not `ui`
      fields back to the read model they display. The scripted break/fix
      (Beat 5) is genuinely enforced and safe as written — this only bites
      if a presenter ad-libs "and it checks your UI fields too." Don't.
- [ ] **`## Delta` blocks are authored, never parsed.** `em diff` diffs
      the `.em` model's structure; it does not understand or verify the
      Added/Modified/Removed prose inside a slice doc's `## Delta`
      section. Beat 11/15 as scripted only claims the structural diff is
      real — never claim "and the Delta block is checked against the
      code," it isn't.
- [ ] **No GA Axon Server connector; this app runs Axon's in-memory event
      store.** Meridian Goods' own `AxonApplicationConfiguration` documents
      this directly: Axon Server support didn't have a compatible
      Maven-Central release at authoring time, so the scaffold runs the
      framework's default in-memory store. Every slice is written exactly
      as it would be against a durable, DCB-capable store — only the
      plugged-in `EventStorageEngine` differs. If asked "does this
      survive a restart in production," say this plainly; don't imply a
      production event store is wired up.
- [ ] **The cross-processor ordering note.** `Request Payment`'s automation
      and the `Payments To Request` projection both react to `Order
      Placed` on independent, pooled-streaming processors with no
      ordering guarantee between them. The decide-path behavior (at most
      one `Payment Requested` per order) is proven by tests that pre-seed
      the read model directly; the real race between the two processors
      racing each other on the same event is not exercised end-to-end by
      any test in this repo. If asked "what happens if the automation
      outruns the projection," say exactly that — it's a real,
      acknowledged architectural gap in this revision, not a solved
      problem, and not the invariant this beat is claiming to prove.
- [ ] **`em coverage` requires `--tests <dir>`** as a mandatory option, not
      an optional flag — script the full command
      (`em coverage meridian-goods.em --tests src/test --strict`), don't
      ad-lib it live or the command errors on stage.
- [ ] **"Does em have a status line / dashboard now?" is a yes, not covered
      by this script.** As of em 1.9.0, `em status` prints exactly that —
      real output on this repo is `8/8 implemented · 20/20 invariants
      covered · 0 open issues · last conformed 8f12ed8 — 11 commits and 0
      slice-PRs behind HEAD` — plus `em freshness` for just the
      conformance clause, `em slice ratify --by "<name>"` for named
      ratification, and `em ci init` for the CI preset. All real, all
      validated against this exact repository ahead of release. None of
      it is in this script because the script is locked to an earlier
      pin — say so plainly rather than improvising a beat live.
- [ ] **`em migrate` is a Q&A asset, not a scripted beat.** If asked "what
      happens to an old two-slice-shape model," have the before/after
      ready: `em migrate <file>` (dry run) reports what it would move,
      `--write` applies it, folding a `processor`/`translation` forward
      into the slice with its command and event and auto-adding the
      `from` clause; a second `--write` is a confirmed no-op
      (idempotent). Don't run this live — it's not part of either script.

## Final pass

- [ ] Run `script.md` beat-by-beat, once, start to finish, on a fresh
      clone, with a stopwatch on Beats 1, 2, and 10 (the three live
      Socratic segments — the only real timing variance in the talk).
- [ ] Run `teaser-5min.md` once on its own, same fresh-clone standard.
- [ ] After the dry run, `git status` in every clone touched — nothing
      tracked should show a diff (the SVG highlight-color rewrite and the
      two local-only frontmatter reverts are the only expected working-tree
      changes, and both get reverted mid-script). No new commits, no
      pushes to `main`.

## Version re-verification (2026-08-27)

All captures in the script were made on em 1.8.0 / em-sdd-bridge 0.4.0 and
re-verified identical on **em 1.8.1 / bridge 0.4.1** (the current releases;
both scaffold-template and error-message fixes shipped), with one known
delta: `em coverage` prints "24 invariant(s) checked" on 1.8.1 (was 25) —
say "**20 distinct invariants, 0 uncovered**" on stage; the raw attribution
count is version-specific.

## Version note (2026-08-28, em 1.9.0)

This script is **not** re-locked to em 1.9.0 — none of its beats are
factually broken by it, but a presenter running against a 1.9.0 clone
should know about four deltas before fielding a question on them:

- **`em coverage` now prints "20 invariant(s) checked"** on Meridian Goods
  (was 24 on 1.8.1) — a second overcount fix, on wrapped continuation
  lines this time. The raw count now equals the distinct count for the
  first time; keep saying "20 distinct invariants, 0 uncovered" on stage,
  it's now also literally what the tool prints.
- **Beat 9b's "exactly 7 read-only tools" is a dated capture, not a
  current fact.** `em-mcp`'s `tools/list` returns 13 tools on 1.9.0
  (`diff`, `glossary`, `changelog`, `conform_scope`, `freshness`, and
  `status` joined the original seven), under a stated MCP-parity
  invariant the project now holds itself to going forward. Don't ad-lib
  "exactly 7" as a current fact if presenting from a 1.9.0 checkout — say
  "seven at the versions this talk is pinned to; the count has grown
  since, under a standing parity rule."
- **A fresh `em skill install --force` on a 1.9.0 clone whose skill
  bundle predates the 6-directory split will not clean up the old
  bundle** — orphaned `reference/`/`templates/` files under
  `.claude/skills/event-modeling/` make `em skill check` report
  mismatches until `em skill sync` runs instead. Real rough edge, found
  validating 1.9.0-pre against `meridian-goods` — don't run `em skill
  install --force` live on an old clone as a demo of the skill bundle;
  use `sync`, or don't touch it live at all.
- **A freshly regenerated Slices table gains a "Ratified by" column** (new
  in 1.9.0, from each doc's `ratifiedBy` frontmatter) — none of Meridian
  Goods' eight slice docs carry that field yet, so every cell reads `—`.
  If `em slice index` gets re-run live during Beat 4 or 16 on a 1.9.0
  build, expect this column to appear with empty cells; that's real
  staleness in the demo repo's own docs, not a rendering bug, and it's
  fine to say so on stage.

None of this is in scope for the locked beats above — it's here so a
Q&A answer doesn't contradict what a curious attendee just checked on
their own laptop.
