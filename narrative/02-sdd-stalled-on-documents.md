# SDD stalled where it started: on documents

Spec-driven development did the hard part — it got teams writing structured
intent before code. Then, in public, it started reporting the same three
symptoms, on repeat, from people actually running it.

**Drift.** François Zaninotto's November 2025 field report found the agent
itself breaking from its own spec: it marked a "verify implementation" task
done "without writing a single unit test" — the spec said one thing, the
agent's actual behavior said another, and nothing in the workflow caught
the gap before a human read the diff by hand. A spec that nothing re-checks
is a spec that can drift silently for as long as nobody looks.

**Ceremony.** Birgitta Böckeler's October 2025 survey on martinfowler.com
tested Kiro on a single small bug and found "the workflow was like using a
sledgehammer to crack a nut" — the requirements document turned the bug
into "4 'user stories' with a total of 16 acceptance criteria," including a
story reading "As a developer, I want the transformation function to handle
edge cases gracefully, so that the system remains robust when new category
formats are introduced." A second, independent report from this August
put the same complaint in plainer terms: treating every SDD phase as
mandatory ceremony instead of a tool you reach for when the task warrants
it "can double documentation overhead" on a project.

**The verdict from the people whose job is watching this stuff.**
Thoughtworks' Technology Radar placed spec-driven development at Assess —
not Trial, not Adopt — in Volume 34 (November 2025), noting the workflows
"remain elaborate and opinionated," and closing on the open question of
whether "handcrafting detailed rules for AI ultimately doesn't scale." One
practitioner's January 2026 postmortem on a failed SDD rollout put it more
bluntly than any analyst would: "This wasn't spec-driven development. It
was vibe specifying. Waterfall with a chatbot." That line is real and
dated, but it's one person's account under a personal handle, not an
industry consensus — worth repeating for how sharply it names the failure
mode, not as a stand-in for a survey.

None of this is a case against structured intent. It's the sound of teams
discovering that a document nobody re-checks behaves exactly like every
other document nobody re-checks: it starts accurate and gets less accurate
every day code moves without it. Drift, ceremony, and "waterfall with a
chatbot" are not three separate diseases. They're three symptoms of one
missing piece.

## The missing piece is a layer, not a template

Here's the diagnosis Act II is built on. An SDD spec — spec-kit's feature
spec, Kiro's requirements.md, Tessl's spec-as-source file — describes
**intended code**. It lives at code's layer, so it inherits code's churn
rate: every refactor, every framework migration, every model-capability
jump washes through it. That's exactly why one entire camp of SDD tooling
(Kiro's own throwaway-after-tasks-complete posture) treats its own spec as
disposable — at that layer, disposability is the honest response to churn,
not a shortcut.

```
BUSINESS PROCESS   — what can happen, in what order, decided by whom
        |  (churns on business time: policy, new products, regulation)
        v
BUILDABLE UNITS    — one command/view/reaction, fully contracted
        |  (a projection — mechanical, cheap to regenerate)
        v
INTENDED CODE      — spec.md, plan.md, framework code, tests
           (churns on technology time: refactors, frameworks, AI shifts)
```

Nothing above describes the *business process* — what can happen, in what
order, decided by whom, under which rule — as something distinct from the
code that happens to implement it today. And because nothing describes it
separately, nothing can check the implementation against it either. The
ceremony inflates because there's no artifact carrying that content
efficiently; the drift goes unnoticed because there's no artifact left to
diff against once the spec's own job is done.

## Takeaway

The gap isn't a better spec format. Sharper acceptance criteria, less
ceremony, a leaner requirements template — none of it touches the actual
hole, which is a layer above specs, describing the business process itself,
plus machinery that keeps that description true after the code ships. That
layer, and that machinery, is where this story goes next.
