# Slicewright

Slicewright is an agentic software development lifecycle built on event
modeling. Agents propose slices of behavior, humans ratify them, and machines
check that the implementation stays true to what was ratified — agents
propose, humans ratify, machines check — using
[`em`](https://www.npmjs.com/package/@milehimikey/em) and
[`em-sdd-bridge`](https://www.npmjs.com/package/em-sdd-bridge) as the
toolset, with the [Meridian Goods](https://github.com/milehimikey/meridian-goods)
demo as the proof artifact.

**Status: early — evidence stated at its exact size.** See
[narrative/09](narrative/09-early-credible-tryable.md) for what's
demonstrated, what's argued from one or two dated data points, and what's
still just argument.

## Read the story

The narrative runs nine beats, one file per beat:

1. [Code got cheap. Correctness of intent didn't.](narrative/01-code-got-cheap.md)
2. [SDD stalled where it started: on documents](narrative/02-sdd-stalled-on-documents.md)
3. [Describe the business process, not the intended code](narrative/03-describe-the-business-process.md)
4. [A ratified slice IS the spec](narrative/04-a-ratified-slice-is-the-spec.md)
5. [Every change is add, change, or remove a slice](narrative/05-change-is-add-change-remove.md)
6. [Two loops: demand and supply](narrative/06-two-loops.md)
7. [The Meridian Goods walkthrough](narrative/07-the-meridian-goods-walkthrough.md)
8. [Governance without ripping anything out](narrative/08-governance-without-ripping-out.md)
9. [Early, credible, tryable today](narrative/09-early-credible-tryable.md)

## Watch it or present it

- [`walkthrough/`](walkthrough/) — the locked, presenter-ready script for the
  Meridian Goods walkthrough (the full ~25-minute version and a 5-minute
  teaser), pinned to `em 1.8.0` / `em-sdd-bridge 0.4.0`.
- [`decks/`](decks/) — two Marp-markdown slide decks derived from that
  script (teaser and full walkthrough), rendered locally with
  `npx @marp-team/marp-cli` — no account or install required.

## The toolset

- [`em`](https://www.npmjs.com/package/@milehimikey/em) — the CLI, MCP
  server, and facilitation skill for event modeling: model, slice, validate,
  export, conform. [Source](https://github.com/milehimikey/em).
- [`em-sdd-bridge`](https://www.npmjs.com/package/em-sdd-bridge) — bridges a
  ratified `em` slice doc straight into a spec-kit-based SDD flow, either as
  a real filesystem symlink or a generated spec with literal invariant
  back-references. [Source](https://github.com/milehimikey/em-sdd-bridge).
- [`em-portal`](https://github.com/milehimikey/em-portal) — **0.1.0, in
  development, not yet on npm.** A separate, fully static site built from
  `em export`/`em status` JSON: a read-only multi-model browser with a
  status-first landing page. Decided and scaffolded, not yet exercised by
  a real pilot — see [narrative/09](narrative/09-early-credible-tryable.md).
- [`em-tracker-bridge`](https://github.com/milehimikey/em-tracker-bridge) —
  **0.1.0, in development, not yet on npm.** The same thin-adapter shape as
  `em-sdd-bridge`, aimed at an org's issue tracker (Linear first) instead of
  an SDD tool: mirrors a ratified slice's lifecycle status into it without
  `em` itself becoming a tracker.

## The proof artifact

[Meridian Goods](https://github.com/milehimikey/meridian-goods) is a small
ecommerce ordering system, modeled and implemented end to end with this
toolset — eight slices, four patterns, one completed conform-and-ratify
cycle with a dated report.

## Try it

This is a pilot invitation, not a product launch with a sales team behind
it. If your team already runs an SDD tool, point `em-sdd-bridge` at one
ratified slice and watch it refuse an unratified one. If you don't, the
whole loop — model, gate, implement, conform, ratify — is the complete
pipeline; there's nothing else to install underneath it. See
[narrative/09](narrative/09-early-credible-tryable.md) for exactly what
we're asking a pilot for.

Ready to actually run one? [`playbook/`](playbook/) is the adoption
playbook: pilot selection criteria, the four roles, a week-by-week 4-week
arc with verified commands, the metrics a pilot commits to collecting, and
the governance mapping for your review board — everything a team that has
never seen `em` needs to start from the playbook alone.
