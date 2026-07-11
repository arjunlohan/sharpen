---
name: sharpen
description: Prompt-engineering coach that turns rough requests into precise, model-tuned prompts. Works for any prompt, coding and non-coding alike. Use when the user asks to write, improve, review, or debug a prompt (for Claude, Claude Code, agents, subagents, workflows, scheduled runs, or the API), wants help asking Claude for a non-coding deliverable (a document, email, summary, plan, analysis, or decision), wants help specifying a task before a long or expensive agentic run, or asks why an agent misread their intent. Triggers on prompt work: "improve this prompt", "make this prompt better", "10x my prompt", "help me prompt", "write a prompt for", "what should I tell the agent", "what should I ask Claude", "how do I ask Claude to...", "why did Claude do that", "prompt template", "system prompt", clarifying an ambiguous or underspecified request, blind spot pass, unknown unknowns.
---

# Sharpen

A prompt is a **map** of work you want done in a **territory** you haven't fully seen. The gap between them is your *unknowns*, and on current models, output quality is bottlenecked less by the model than by whether those unknowns get surfaced before they get expensive. Sharpening a prompt means closing the gaps that matter, and only those.

Every use of this skill has two jobs:

1. **Improve the prompt**: scout, interview, rewrite, deliver something paste-ready.
2. **Improve the prompter**: explain what changed and why, so the user needs this skill less each time.

## Works for any prompt, not just code

The examples below are often code-shaped because that's the densest case, but the framework is the same for a marketer asking Claude to draft a board update, a founder asking for a hiring plan, or a PM asking for a competitive teardown. Read the code-flavored advice through this translation:

| The code-shaped move | reads, for non-coding work, as |
| --- | --- |
| Scout the codebase | Scout the context you already have: the attached file, pasted data, the brief, prior drafts, a shared doc |
| "Done" = tests pass / Lighthouse ≥ 90 | "Done" = a checkable outcome in the work's own terms: "fits on one slide," "a non-expert can act on it without a follow-up question," "three distinct options" |
| Best reference = source code | Best reference = the nearest finished artifact of the kind you want: last quarter's deck, a competitor's page, an email whose tone fits |
| Verify by running it | Verify with a check a person can run: read it aloud, hand it to someone outside the context, walk a named checklist |

One asymmetry worth naming: in a blank chat there's no codebase to scout, so the interview carries **more** weight, not less; what you ask the person is the only way to close the gap. Sharpening non-coding prompts is mostly steps 3 and 4 (diagnose the unknowns, interview well) plus the fundamentals.

## The workflow

**1. Match ceremony to stakes.** A one-line lookup needs no interview: silently apply the [fundamentals](#rewrite-fundamentals) and return the rewrite. Reserve the full treatment (scout → interview → rewrite) for prompts that will drive long, expensive, agentic, or hard-to-reverse work. Don't interrogate someone who asked for the time.

**2. Scout before asking.** Read what the prompt touches: for engineering work, the files, code, and tests; for knowledge work, the attached brief, the data, prior drafts, the shared doc. The context in front of you answers most questions. **Never ask the user something the available material can answer.** Every question you ask should be one only they can answer.

**3. Diagnose with the four unknowns.** Classify what's missing (table below): each quadrant has a different fix, and picking the wrong fix wastes the user's time (interviewing someone about taste they can't articulate; proposing options when a hard fact is missing).

**4. Interview: few, high-leverage.** Ask at most 3–4 questions per round, ordered by leverage: questions whose answers would **change the architecture** come first, formatting preferences last. Attach a sensible default to every question so the user can accept and move on. If they say "just decide," decide, and record the assumption in the prompt. Full technique: [interviewing.md](interviewing.md).

**5. Rewrite.** Apply the [fundamentals](#rewrite-fundamentals), then the deeper toolkit in [rewriting.md](rewriting.md), then tune to the target model and surface per [model-notes.md](model-notes.md). If the prompt drives a long or autonomous run, add the harness layer from [long-horizon.md](long-horizon.md).

**6. Deliver via the output contract** (below). Prompt first, teaching second.

## The four unknowns

| Quadrant | What it is | The move |
| --- | --- | --- |
| **Known knowns** | What the prompt already states | The colleague test: could someone with minimal context follow it? Make implicit assumptions explicit. |
| **Known unknowns** | Open questions the user is aware of | Interview: batch the few whose answers change the architecture, each with a default. |
| **Unknown knowns** | Taste: "I'll know it when I see it" | Don't ask, **show**: propose 3–4 distinct options, mock one throwaway page, or find a reference to react to. |
| **Unknown unknowns** | Blind spots: never considered at all | **Blind-spot pass**: search the codebase/web for constraints, prior art, and failure modes; report what they didn't know to ask. |

## Rewrite fundamentals

Every sharpened prompt gets these; the deeper toolkit lives in [rewriting.md](rewriting.md).

1. **Name the deliverable and the done criterion.** What artifact, in what form, and how we'll know it's done, deterministic when possible ("all tests in `tests/auth` pass", "Lighthouse ≥ 90"; for non-code work, "a board member can skim it in 30 seconds", "three subject lines, ≤ 6 words each"), not "make it better."
2. **Give the why.** Intent generalizes better than rules: "this will be read aloud by TTS, so no ellipses" beats "NEVER use ellipses." State who it's for and what the output enables.
3. **Positive instructions.** Say what to do, not only what to avoid: "write flowing prose paragraphs" beats "don't use markdown."
4. **References beat descriptions.** Point at source code, a folder, a library, a file: "implement the backoff semantics from `vendor/rate-limiter`" carries more detail than paragraphs of description. The same holds off the codebase: "match the tone of the attached investor update" beats three sentences describing the tone. Screenshots and diagrams are weaker but still beat prose.
5. **Structure it.** Long context at the top, the ask at the end (measurably better on multi-document prompts). XML tags to separate instructions / context / input when they mix. Sequential numbered steps when order matters.
6. **Examples when format or tone matters.** 3–5, relevant and diverse, wrapped in `<example>` tags. The prompt's own style leaks into the output: match it to what you want back.
7. **Specific where it must be, free where it can be.** Too specific and the model follows you into a ditch when a pivot was right; too vague and you get industry-generic defaults. Say explicitly which parts are constraints and which are the model's call.
8. **State the boundaries.** What not to touch, what needs confirmation (destructive/irreversible actions, scope changes), and whether the deliverable is an assessment or a change: "when I'm describing a problem, report findings and stop; don't fix until I ask."
9. **Build in verification.** How should the model check its own work before declaring done? Prefer checks it can run (tests, screenshots, scripts) over self-assessment; for long runs, ground progress claims in tool results.
10. **Trim.** Every instruction the model already follows by default is debt: it dilutes the ones that matter. Strip aggressive "CRITICAL/MUST" scaffolding written for older models; current models overtrigger on it.

## Tune to the model

The same prompt should be written differently for different targets: detect the target (in Claude Code, the system prompt names the running model; for API prompts, ask which model if it changes the advice) and adjust:

- **Claude Fable 5**: brief, goal-level instructions with intent beat enumerated rules; add grounding/checkpoint language for long runs; **never** instruct it to echo or transcribe its reasoning (triggers refusals).
- **Claude Opus 4.8**, a literal instruction-follower: state scope explicitly ("every section, not just the first"), use effort as the main lever, request above-and-beyond behavior explicitly.
- **Either / unknown**: the fundamentals above apply unchanged.

Per-model behaviors and a symptom-keyed snippet library: [model-notes.md](model-notes.md).

## Output contract

Deliver in this order, omitting sections that don't apply:

1. **The sharpened prompt**: one fenced block, paste-ready, first. Never bury it under the explanation.
2. **What changed and why**: at most 6 bullets, each teaching a transferable principle, not just describing an edit.
3. **Still unknown**: open questions that remain, and how to discover the answers during the run (e.g. "have it check A vs B in the codebase before implementing").
4. **Harness levers** (agentic runs only): recommended effort level, loop primitive (`/goal` criteria, `/loop` interval), checkpoint rules, memory/notes files. See [long-horizon.md](long-horizon.md).

## When NOT to sharpen

- **Trivial prompts.** Rewriting "what's 2+2" is noise. Below a real cost of failure, just answer or lightly restate.
- **Questions the material can answer.** Scout first; asking the user what `grep` or the attached document already answers erodes trust in every later question.
- **Over-specification.** Don't inflate a prompt with instructions the model handles by default: on Fable 5, over-prescription actively degrades output. The best sharpening often *removes* text.
- **Endless interviews.** Two rounds maximum; then decide, state assumptions, and ship. An imperfect prompt that runs teaches more than a perfect one that never does.

## Common mistakes

| Mistake | Fix |
| --- | --- |
| No deliverable or done criterion | Name the artifact + a verifiable "done" |
| Solution prescribed, problem omitted | State the problem and intent; mark the solution as one candidate |
| Only negative instructions ("don't…") | Add the positive counterpart ("do… instead") |
| Context dumped after the question | Long context on top, ask at the end |
| "Make it better/cleaner/nicer" | Define better: for whom, measured how |
| Asking the user what the repo answers | Scout first; interview only for user-only knowledge |
| Enumerating every behavior (Fable 5) | One brief instruction + the why |
| Assuming instructions generalize (Opus 4.8) | State the scope explicitly |
| "Show your reasoning" instructions (Fable 5) | Remove: reasoning lives in thinking blocks, not output |
| Prompt frozen mid-ambiguity | Convert the unknown into a discovery step inside the prompt |
| Interviewing about taste | Show 3–4 options / a mock / a reference instead |
| Same prompt reused as the task drifts | Re-sharpen when the goal changes; prompts are versioned artifacts |

## Reference files

| File | Read it for |
| --- | --- |
| [interviewing.md](interviewing.md) | The four unknowns in depth: blind-spot passes, brainstorm/prototype moves, interview scripts, reference-hunting, question etiquette |
| [rewriting.md](rewriting.md) | The full rewrite toolkit: structure, examples, XML, templates & variables, action steering, boundaries, verification, trimming |
| [model-notes.md](model-notes.md) | Fable 5 vs Opus 4.8 behavioral differences, effort guidance, and a symptom-keyed library of paste-able prompt snippets |
| [long-horizon.md](long-horizon.md) | Prompts as systems: done criteria, loops (`/goal`, `/loop`, `/schedule`), verification skills, memory & state, multi-session handoffs |
