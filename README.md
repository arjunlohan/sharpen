# sharpen

**A prompt-engineering coach, packaged as an Agent Skill** — for Claude Code, Cursor, Codex, Gemini CLI, and other skill-aware assistants.

Most people don't know how to prompt — and the better models get, the more output quality is bottlenecked by the prompt, not the model. `sharpen` teaches your assistant to take a rough request and make it dramatically better: **scout** the codebase before asking anything, **interview** you for only the answers that would change the architecture, **rewrite** the prompt with intent, structure, and verifiable done-criteria, and **tune** it to the model that will run it (Claude Fable 5 vs. Claude Opus 4.8 behave differently — the skill adjusts). Along the way it explains what changed and why, so you get better at prompting, not just this prompt.

**See it work: [the showcase](https://arjunlohan.github.io/sharpen/)** — the same rough prompts sharpened for Claude Fable 5 vs. Claude Opus 4.8 side by side, plus live A/B pairs where the same model built the same request twice and only the prompt differed.

## Install

```sh
npx skills add arjunlohan/sharpen        # project-scoped
npx skills add arjunlohan/sharpen -g     # global: available in every project
```

Once installed it **auto-triggers** on prompt work ("improve this prompt", "write a prompt for…", "why did the agent do that?"), or invoke it manually with `/sharpen`. Without installing, you can also point any assistant at [`skills/sharpen/SKILL.md`](skills/sharpen/SKILL.md).

## What it does

1. **Matches ceremony to stakes** — a one-liner gets a quick rewrite; a prompt that will drive hours of agentic work gets the full treatment.
2. **Scouts before asking** — reads the files/code/docs the prompt touches; never asks you what `grep` can answer.
3. **Diagnoses with the four unknowns** — known knowns (colleague test), known unknowns (interview), unknown knowns (show options, don't ask about taste), unknown unknowns (blind-spot pass).
4. **Interviews with defaults** — ≤4 questions per round, architecture-changing first, each with a recommended default.
5. **Rewrites and tunes** — deliverable + done criteria, the why, structure, examples, boundaries, verification; then model-specific tuning and, for long runs, harness levers (loops, memory, checkpoints).
6. **Delivers prompt-first** — the sharpened prompt in one paste-ready block, then what changed and why, then what's still unknown.

## What's inside

| File | Covers |
| --- | --- |
| [`SKILL.md`](skills/sharpen/SKILL.md) | The spine: the workflow, the four-unknowns frame, rewrite fundamentals, output contract, common mistakes |
| [`interviewing.md`](skills/sharpen/interviewing.md) | Finding unknowns — blind-spot passes, brainstorm/prototype moves, interview scripts, references, question etiquette |
| [`rewriting.md`](skills/sharpen/rewriting.md) | The rewrite toolkit — structure, XML, examples, templates & variables, action steering, boundaries, verification, trimming |
| [`model-notes.md`](skills/sharpen/model-notes.md) | Claude Fable 5 vs. Claude Opus 4.8 behavioral differences, effort guidance, and a symptom-keyed snippet library |
| [`long-horizon.md`](skills/sharpen/long-horizon.md) | Prompts as systems — done criteria, loops (`/goal`, `/loop`, `/schedule`), verification skills, memory & state, handoffs |

## Model-aware by design

The same instructions land differently on different models, so `sharpen` detects the target and adjusts:

- **Claude Fable 5** — brief, goal-level instructions with the *why*; grounding and checkpoint clauses for long runs; never "show your reasoning" (it triggers refusals); over-prescriptive prompts actively degrade output, so sharpening often means **removing** text.
- **Claude Opus 4.8** — a literal instruction-follower: scope stated explicitly, effort as the primary lever, above-and-beyond requested rather than assumed, design directions specified or proposed-first.
- **Other/unknown targets** — the cross-model fundamentals still apply unchanged.

## See also

Sibling of [arjunlohan/finesse](https://github.com/arjunlohan/finesse) — the design-engineering counterpart: motion, micro-interactions, and interface polish.

## License

[MIT](LICENSE) © Arjun Lohan
