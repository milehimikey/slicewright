# Code got cheap. Correctness of intent didn't.

Something changed in software delivery in the last two years that isn't
marketing: writing code got cheap. An agent that used to need careful
prompting to produce a working function now produces the function, the
tests, and a first pass at the wiring, often before you've finished
describing what you wanted. The bottleneck did not disappear — it moved.
It used to sit at "can we write this." Now it sits at "does anyone actually
know, precisely, what this should do."

You don't have to take that on faith. By the end of 2025, every major
coding-agent vendor had shipped a named spec-driven-development flavor:
[GitHub's Spec Kit](https://github.blog/ai-and-ml/generative-ai/spec-driven-development-with-ai-get-started-with-a-new-open-source-toolkit/)
(September 2025), [AWS's Kiro](https://kiro.dev/blog/introducing-kiro/)
(July 2025, GA May 2026), and
[Tessl's framework](https://tessl.io/blog/tessl-launches-spec-driven-framework-and-registry)
(September 2025), alongside the independent
[OpenSpec](https://github.com/Fission-AI/OpenSpec) project. Four
independently-built tools, four different companies, converging on the same
diagnosis inside twelve months: don't hand the agent a raw prompt, hand it a
structured artifact first.

That convergence is the whole point of this beat, and it's worth being
precise about what it does and doesn't prove. It doesn't prove any one of
these tools got the artifact right — Act I's next beat is about where they
didn't. What it proves is narrower and already settled: **the industry
stopped arguing about whether agents need something more than a prompt.**
GitHub's own framing put it plainly — Spec Kit exists to "provide a
structured process to bring spec-driven development to your coding agent
workflows," built for exactly the vendor-agnostic set of tools teams already
use (GitHub Copilot, Claude Code, Gemini CLI, among them). Kiro's pitch is a
"simplified developer experience for working with AI agents," anchored on a
requirements → design → tasks workflow before a single line of
implementation. Tessl frames its spec as giving agents "the information they
need about both what and how you want them to build, bolstered by tests and
hard guardrails." Different vocabulary, same shape: description first,
generation second.

## What this means for the rest of this story

If you build software with agents today, you already act on this belief,
whether or not you use any of the four tools above. You write a ticket with
more detail than you used to, because you've noticed that an underspecified
prompt gets you a plausible-looking wrong answer faster than a human ever
would have. You've started keeping a `spec.md` or a `requirements.md`
somewhere, even if it's informal. That instinct is correct, and it isn't
new — it's the same instinct that produced design docs and RFCs before any
agent existed. What's new is that the industry now ships tooling that
assumes it.

So this story does not open by trying to convince you that agents need a
structured artifact in front of them instead of a bare prompt. That
argument is over, and four separate companies spent 2025 proving it by
shipping product, not by publishing a manifesto. The interesting question —
the one the rest of this story is actually about — is what that artifact
should *describe*. A spec that describes the code you're about to write is
one answer. It is also, as it turns out, the answer that's already visibly
straining under its own weight, in public, in the words of the people
running it in production. That's the next beat.

None of this required a single named authority to declare it true. It
required four competitors — none of whom benefits from the others' success
— to independently spend real engineering budget on the same bet inside the
same twelve-month window. That's a stronger signal than any one company's
roadmap slide, and it's the reason this story doesn't spend more time
convincing you of it than it just did.

## Takeaway

You already believe a structured artifact must precede agent work. That
argument is over — every major vendor already built one. What's still open
is what the artifact is *of*.
