# Rewriting: the toolkit

The interview found what was missing; this file is how to encode it. Apply on top of the fundamentals in [SKILL.md](SKILL.md); tune the result per [model-notes.md](model-notes.md).

## Deliverable & done

The two sentences most prompts are missing:

- **The deliverable:** the artifact, its format, its audience, and its **viewing context** (viewport, embed, projection) when one dominates. "A single self-contained HTML page for the exec review," not "something showing the data."
- **The done criterion:** verifiable, ideally deterministic. "All tests in `tests/auth` pass and `npm run lint` is clean," "the diff touches only `src/billing/`," "Lighthouse ≥ 90." For non-code work it's just as checkable: "a one-page brief a non-expert can act on without a follow-up question," "5 headline options ≤ 8 words each," "every claim cites a source." A checkable "done" is what lets the model iterate instead of stopping at plausible, and what makes goal-based loops possible ([long-horizon.md](long-horizon.md)).
- **A resilience floor for anything visual:** even when one context dominates ("built for a TV"), keep "holds at 360px and 1280px" in the done criteria. Artifacts get judged in places they weren't designed for: embedded, split-screened, screenshotted, and a prompt that optimizes hard for one viewport buys a layout that shatters in the others unless you say so.

If you want above-and-beyond, **ask for it**: "include as many relevant features and interactions as possible; go beyond the basics", rather than hoping the model infers ambition from a bare noun phrase.

## Intent and motivation

State *why*, *who for*, and *what it enables*. Models generalize from reasons: "responses are read aloud by TTS, so avoid ellipses" covers a hundred cases the bare rule doesn't. The frame that consistently pays off:

```text
I'm working on [the larger task] for [who it's for]. They need [what the output enables].
With that in mind: [request].
```

## Structure

- **Long context first, ask last.** On multi-document prompts, putting the query at the end improves quality by up to ~30%. Order: documents/data → instructions → examples → the ask.
- **XML tags** whenever instructions, context, examples, and input mix: `<instructions>`, `<context>`, `<input>`, `<example>`. Consistent names; nest naturally (`<documents>` → `<document index="n">` → `<source>` + `<document_content>`).
- **Quote-grounding for long documents:** have the model first extract relevant quotes into `<quotes>`, then work from those: it cuts through the noise of a 100-page input.
- **Numbered steps** when order or completeness matters; prose when it doesn't.
- **A role, in one sentence**, when tone or domain focus matters: "You are a senior Python reviewer." Skip role-play theatrics.

## Examples

The most reliable lever for format, tone, and structure:

- 3–5 examples, wrapped in `<example>` / `<examples>` tags so they can't be mistaken for instructions.
- **Relevant** (mirror the real use case), **diverse** (cover edge cases; vary what shouldn't be learned as a pattern).
- The prompt's own formatting leaks into output: heavy markdown in → heavy markdown out. Match your prompt's style to what you want back.
- Examples can carry *reasoning style* too: show worked reasoning inside the example when you want the model to reason that way.

## Output formatting

- **Positive spec:** "write flowing prose paragraphs" beats "no markdown." Tell it what the output *is*.
- **Format indicators:** "put the analysis in `<analysis>` tags" is self-enforcing.
- **Structured outputs / tool schemas: not prefills.** Prefilled assistant turns are gone on current models; use structured outputs for JSON, enum tool fields for classification, and retries for conformance.
- For "respond directly without preamble," say exactly that, and strip any survivors in post-processing rather than fighting for the last 1%.

## Templates & variables (API prompts)

Split fixed from variable content; mark variables `{{like_this}}` and wrap them in XML tags. This buys consistency, testability (swap only the variable part), and versioning (the template is the artifact under version control). A sharpened API prompt should *be* a template: deliver it with its variables named.

## Action steering

Verbs decide whether the model suggests or does:

- "Can you suggest changes?" → suggestions. "Change this function…" / "Make these edits…" → edits.
- To set a standing default, pick one and state it:

```text
By default, implement changes rather than only suggesting them. If intent is unclear,
infer the most useful likely action and proceed, using tools to discover missing
details instead of guessing.
```

```text
Do not jump into implementation unless clearly instructed to make changes. When intent
is ambiguous, default to research and recommendations; only edit when explicitly asked.
```

- **Parallelize when independent:** "when multiple tool calls have no dependencies between them, make them in parallel": near-100% parallel usage when stated.

## Boundaries & safety rails

For any prompt that can change state:

```text
You may freely take local, reversible actions (edit files, run tests). For actions that
are destructive, hard to reverse, or visible to others (deleting branches, force-push,
posting/sending anything external, schema changes), ask first. Never bypass safety
checks (--no-verify) or discard unfamiliar files to get unblocked.
```

And separate *assessment* from *action*: "When I'm describing a problem or thinking out loud, the deliverable is your assessment: report findings and stop; don't fix until I ask."

## Quality rails (agentic coding)

Symptoms → clauses, use only what the task risks:

- **Over-engineering:** "Don't add features, refactors, or abstractions beyond the ask. No error handling for scenarios that can't happen; validate only at system boundaries. The right amount of complexity is the minimum for the current task."
- **Test-gaming:** "Write a general-purpose solution for all valid inputs, not just the test cases. Tests verify correctness; they don't define the solution. If a test is wrong or the task infeasible, say so rather than working around it."
- **Hallucinated claims about code:** "Never speculate about code you haven't opened. If a file is referenced, read it before answering."
- **File litter:** "If you create temporary scripts or scratch files for iteration, delete them at the end."

## Verification

End the prompt with how the work gets checked, in descending strength:

1. **Runnable checks**: tests, linters, a build, a screenshot diff, a script. "Before declaring done, run X; if it fails, fix and rerun."
2. **Fresh-context review**: a separate verifier subagent against the spec (beats self-critique, which inherits the author's blind spots).
3. **Self-check against named criteria**: "verify your answer against [criteria] before finishing." Weakest, still better than nothing.

When nothing is runnable (writing, analysis, planning), substitute a **concrete human check** rather than dropping verification: read it aloud, hand it to someone outside the context ("would a new hire act correctly on only this?"), or walk a named checklist. A specific human check beats "review it" the same way a passing test beats "make sure it works."

For long runs, add grounded reporting: *"Before reporting progress, audit each claim against a tool result from this session; if something isn't verified, say so explicitly."*

## Trimming: the underrated half

Sharpening is subtraction as often as addition:

- **Delete instructions the model already follows.** Each stale rule dilutes the live ones. Current models need far less scaffolding than the prompts written for their predecessors.
- **De-escalate.** "CRITICAL: you MUST…" written to fix undertriggering on an old model causes overtriggering now. Restate as plain guidance: "Use this tool when…".
- **Kill contradictions.** "Be thorough" + "be brief" forces the model to pick; you pick instead.
- **One instruction, one home.** Repetition reads as emphasis and skews behavior.
- **Never instruct reasoning-echo.** "Show your thinking / explain your reasoning step by step in the response" belongs to an older era: on Fable 5 it can trigger refusals, and on every current model thinking happens in thinking blocks. Ask for a *justified answer*, not transcribed reasoning. (See [model-notes.md](model-notes.md).)
