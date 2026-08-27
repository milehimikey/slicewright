# Decks

Two Marp-markdown presentation decks derived from the locked
`walkthrough/` script, pinned to `em 1.8.0` / `em-sdd-bridge 0.4.0`:

- [`teaser-5min.md`](./teaser-5min.md) — ~12 slides, the compressed
  4-beat teaser arc.
- [`full-walkthrough-25min.md`](./full-walkthrough-25min.md) — ~33
  slides, the full walkthrough beat by beat: one section slide per
  act, one slide per beat with its exact invocation, trimmed real
  capture, and "proves" takeaway; the two-loop honesty table verbatim;
  the four governance pillars; presenter notes carrying narration cues
  and LIVE/PRE-STAGED markers from the script.

Both decks use Marp's built-in `default` theme — no custom fonts or
assets.

## Rendering

Requires no local install; both targets fetch
[`@marp-team/marp-cli`](https://github.com/marp-team/marp-cli) via
`npx` on demand.

```
cd decks
make html   # renders both decks to .html
make pdf    # renders both decks to .pdf
make        # both (default target)
make clean  # removes rendered output
```

Or render one deck directly:

```
npx -y @marp-team/marp-cli teaser-5min.md -o teaser-5min.html
npx -y @marp-team/marp-cli full-walkthrough-25min.md -o full-walkthrough-25min.pdf --allow-local-files
```

Rendered output (`*.html`, `*.pdf`) is gitignored — only the source
`.md` files are committed. Regenerate locally before presenting.

## Presenter notes

Both files carry Marp `<!-- notes -->` HTML comments per slide with
narration cues and LIVE/PRE-STAGED markers, matching
`../walkthrough/script.md` and `../walkthrough/teaser-5min.md`. Marp's
presenter view (`--preview` or the exported HTML's speaker window)
surfaces these; open the export with a Marp-aware viewer to see them
rather than reading the raw HTML source.
