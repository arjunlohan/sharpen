# Long-horizon prompts: the prompt as a system

Past a certain run length, a prompt stops being a request and becomes a **system spec**: it has to define done, verification, memory, checkpoints, and state, because nobody is watching in real time. This file covers what to add when the sharpened prompt will drive hours of autonomous work, a recurring loop, or a multi-session project.

## Done criteria: the load-bearing sentence

Vague "done" is where long runs fail: the model either stops at plausible (too early) or polishes forever (too late). Make done **deterministic and checkable**:

- Counts and thresholds: "all 200 tests pass," "Lighthouse ≥ 90," "zero `grep -r TODO src/` hits."
- Exhaustion conditions: "every report in the queue is triaged, actioned, and responded to."
- Explicit turn/attempt caps as the pressure valve: "stop after 5 tries and report what's blocking."

When the harness supports it, hand the stop condition to a **goal loop** (`/goal` in Claude Code): an evaluator checks the condition each time the model tries to stop and sends it back until the goal is met or the cap is hit. This removes "is this good enough?" from the model's own judgment: the direct antidote to premature completion.

## Pick the loop, not just the prompt

Match the primitive to how the work arrives:

| Loop | You hand off | Use when | Reach for |
| --- | --- | --- | --- |
| Turn-based | The check | Exploring or deciding; short tasks | A verification skill (below) |
| Goal-based | The stop condition | Done is verifiable | `/goal` + turn cap |
| Time-based | The trigger | Work arrives on a schedule / external systems change | `/loop` (local), `/schedule` (cloud) |
| Proactive | The prompt itself | Recurring, well-defined streams (triage, upgrades) | schedule + goal + workflows + auto mode |

Token hygiene: match the interval to how often the watched thing actually changes; pilot workflows on a small slice before a large run; push deterministic steps into scripts the model runs instead of re-reasoning each time.

## Verification the model can run

The more quantitative the check, the better the model self-verifies. Two moves:

1. **Encode your manual checks as a skill.** If a human would open the page, click the control, and check the console, write that as a `verify-*` skill the loop must pass before declaring done: "never report a UI change complete on a successful edit alone; start the dev server, interact with the change, screenshot before/after, zero new console errors; if any step fails, fix and rerun from step 1."
2. **Fresh-context review.** A second agent (or verifier subagent) reviewing against the spec beats self-critique: it isn't invested in the author's choices. For scheduled pipelines: main agent loops until verification passes → opens a PR → a second agent reviews → the human only decides what to merge.

And for anything the model reports upward, grounding (snippet **S3** in [model-notes.md](model-notes.md)): every progress claim audited against a tool result.

## State, memory, and notes

Long work needs writable state, and the prompt should name where it lives:

- **Structured state for structured facts**: `tests.json` with per-test status; the model tracks schema better than prose. Pair with: "removing or editing tests is unacceptable: it hides missing functionality."
- **Freeform notes for narrative**: `progress.txt`: what was done, what's next, what's weird.
- **A deviations log during implementation**: "keep `implementation-notes.md`; if an edge case forces a deviation from the plan, pick the conservative option, log it under Deviations, and keep going." This is how unknowns discovered mid-run get captured instead of silently absorbed.
- **Lessons across runs**: one lesson per file, why included; update rather than duplicate; delete when wrong. Bootstrap by mining past sessions for recurring corrections.
- **Git as the state machine**: commits are checkpoints and the log is the run's history; instruct regular commits with meaningful messages.

## Multi-session and fresh-context handoffs

Plans made in one session are executed best in a **new** one: fresh context, carrying only the distilled artifacts.

- **First session:** scope, interview, spec, prototype, implementation plan (lead the plan with the decisions most likely to change: data models, interfaces, UX, and bury the mechanical parts).
- **Handoff:** start a new session; pass the artifacts (spec, prototype, notes) in the first message.
- **Fresh-start discovery prompt**: be prescriptive about re-orientation: "Run `pwd`; you may only touch this directory. Review `progress.txt`, `tests.json`, and the git log. Re-run the integration test before implementing anything new."
- **Encourage full use of the window:** "It's fine to spend your entire output context on this: just don't run out with significant uncommitted work."
- If the harness compacts automatically, say so: otherwise the model may wrap up early to be safe (snippet **S6**).

## After the run: close the loop

- **Explainers for buy-in:** "package the prototype, spec, and implementation notes into a single doc I can drop in Slack: lead with the demo."
- **Quizzes for understanding:** after a long autonomous session, the diff understates what happened. "Give me a report on the changes: context, intuition, what was done, with a quiz at the bottom that I must pass." Don't merge until you pass; failing the quiz is the cheapest possible integration test of *your* mental model.
- **Feed lessons forward:** anything that surprised you becomes a lesson file or a prompt/skill edit, so the next run starts smarter. When one result misses the bar, fix the *system* (prompt, skill, verification), not just the instance.
