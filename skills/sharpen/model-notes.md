# Model notes — tuning the prompt to its target

The same instructions land differently on different models. Detect the target, then adjust. Anchored on the current pair — **Claude Fable 5** and **Claude Opus 4.8** — with a symptom-keyed snippet library at the end. When the models change, re-verify the specifics here against the current model docs.

## Detect the target

- **In Claude Code**, the system prompt names the running model — read it; don't guess.
- **For API / product prompts**, ask which model will run it *only if* the answer changes the advice (it usually does for effort, verbosity, and long-run scaffolding).
- **Unknown or mixed targets:** write to the shared fundamentals in [SKILL.md](SKILL.md) and skip model-specific snippets — they're tuning, not table stakes.

## Claude Fable 5

Built for long-horizon, ambiguous, end-to-end work — the teams getting the most from it assign the *hardest* problems, not the routine ones. Prompting implications:

**Write less, mean more.** Instruction-following is strong enough that one brief instruction with the *why* replaces an enumerated list of behaviors. Skills and prompts written for prior models are often too prescriptive and **degrade** output — trim before you add. Fable 5 will also sensibly update over-prescriptive guidance based on what it learns mid-task; let it.

**Never instruct reasoning-echo.** Prompts that tell the model to echo, transcribe, or explain its internal reasoning as response text can trigger the `reasoning_extraction` refusal. Audit for "show your thinking" instructions and remove them; reasoning visibility comes from thinking blocks, and mid-run visibility from a send-to-user tool.

**Longer turns are normal.** Hard tasks run many minutes; autonomous runs, hours. Prompt for action over deliberation when that's wrong for the task (snippet **S1**), and check on long runs asynchronously rather than blocking.

**Effort:** `high` is the default for most work; `xhigh` for the most capability-sensitive; `medium`/`low` remain strong for routine tasks (often above prior models' best). If output is right but slow, lower effort before rewriting the prompt.

**Long runs need three clauses** — grounding (S3), checkpoints (S4), and, for unattended pipelines, the autonomy reminder (S5). In testing, grounding alone nearly eliminated fabricated status reports.

**Subagents:** dispatches parallel subagents readily and manages long-running ones well. Say when delegation is appropriate, prefer async over blocking, and reuse long-lived subagents for context-cache wins (S7).

**Memory compounds.** Fable 5 is particularly good at recording and applying lessons across runs — give it a place to write notes (S8), and bootstrap it by having the model mine previous sessions for recurring themes.

**Don't surface context countdowns.** A remaining-token display makes it wrap up early; hide it, or add the reassurance snippet (S6).

## Claude Opus 4.8

Strong long-horizon agentic work, knowledge work, vision; a **literal, precise instruction-follower**. Prompting implications:

**Literalism is the feature.** It won't silently generalize one instruction to neighbors or infer requests you didn't make. State scope explicitly: "apply this formatting to **every** section, not just the first." Great for pipelines and tuned prompts; write accordingly.

**Effort is the primary lever.** `xhigh` for coding and agentic work; ≥`high` for anything intelligence-sensitive; `max` sometimes helps but can overthink. It respects low settings *strictly* — under-thinking at `low`/`medium` is fixed by raising effort, not by prompting around it. At `max`/`xhigh`, give a large output-token budget (start ~64k).

**Verbosity calibrates to the task.** Short answers on lookups, long on open-ended analysis. If a product needs fixed verbosity, say so — and positive examples of the desired concision beat "don't over-explain."

**Reasoning over tools.** It favors reasoning to tool calls; raise effort or explicitly describe when/why to use a tool if it's under-searching.

**Fewer subagents by default.** Opposite of Fable 5 — give explicit spawn guidance both ways: "don't spawn a subagent for work you can do directly; do fan out when reading many files or working independent items in the same turn."

**Above-and-beyond is opt-in.** "Create a dashboard" gets a dashboard; "…include as many relevant features and interactions as possible; go beyond the basics" gets the impressive one.

**Design house style is persistent.** Warm cream backgrounds, serif display type, terracotta accents — beautiful for editorial, wrong for dashboards/fintech/dev tools, and generic nudges ("don't use cream") just swap in a different fixed palette. Either specify a concrete alternative (palette hexes, typeface, radius, motion) or have it **propose 3–4 distinct directions first** and build only the picked one (S9).

**Code-review prompts: split coverage from filtering.** It follows "only report high-severity" so faithfully that measured recall drops. Ask for everything with confidence + severity attached, and filter downstream (S10).

**Interactive coding:** it reasons more after each user turn — front-load the task, intent, and constraints in the first message; fewer, richer turns beat drip-fed clarifications.

## Either model — quick contrast

| Dimension | Fable 5 | Opus 4.8 |
| --- | --- | --- |
| Instruction style | Brief + intent; over-prescription degrades | Literal + explicit scope; won't generalize unstated |
| Effort default | `high` (xhigh for hardest) | `xhigh` coding/agentic; ≥`high` sensitive work |
| Subagents | Eager — guide *when*, go async | Conservative — guide *when to spawn* |
| Long runs | Ground claims, checkpoints, memory | Strong; remove forced-status scaffolding |
| Reasoning in output | Never instruct — refusal risk | Don't instruct; use thinking blocks |
| Design output | — | Persistent house style; specify or propose-first |

## Snippet library — keyed by symptom

Paste-able clauses keyed by symptom. Use the one that matches what you're actually seeing; don't stack them preemptively.

**S1 — Overplanning / narrating instead of doing** *(Fable 5, higher effort)*
```text
When you have enough information to act, act. Do not re-derive facts already established
in the conversation, re-litigate a decision the user has already made, or narrate options
you will not pursue. If you are weighing a choice, give a recommendation, not a survey.
```

**S2 — Unrequested cleanup / gold-plating** *(both, higher effort)*
```text
Don't add features, refactor, or introduce abstractions beyond what the task requires.
A bug fix doesn't need surrounding cleanup. Do the simplest thing that works well; don't
design for hypothetical future requirements. Only validate at system boundaries.
```

**S3 — Fabricated or rosy status reports** *(long runs)*
```text
Before reporting progress, audit each claim against a tool result from this session.
Only report work you can point to evidence for; if something is not yet verified, say
so explicitly. If tests fail, say so with the output; if a step was skipped, say that.
```

**S4 — Stops too often / asks permission it doesn't need**
```text
Pause for the user only when the work genuinely requires them: a destructive or
irreversible action, a real scope change, or input only they can provide. If you hit
one of these, ask and end the turn — otherwise keep going.
```

**S5 — Ends the turn on a promise ("I'll now run X")** *(autonomous pipelines)*
```text
You are operating autonomously; the user cannot answer questions mid-task. For
reversible actions that follow from the original request, proceed without asking.
Before ending your turn, check your last paragraph: if it is a plan, a question, or a
promise about work not yet done, do that work now with tool calls. End only when the
task is complete or blocked on input only the user can provide.
```

**S6 — Wraps up early citing context limits** *(Fable 5, visible token counters)*
```text
You have ample context remaining. Do not stop, summarize, or suggest a new session on
account of context limits. Continue the work.
```

**S7 — Subagent steering** *(Fable 5: when; Opus 4.8: whether)*
```text
Delegate independent subtasks to subagents and keep working while they run. Don't spawn
a subagent for work you can complete directly; do fan out when reading many files or
working independent items. Intervene if a subagent goes off track.
```

**S8 — Repeats old mistakes across sessions** *(memory)*
```text
Store one lesson per file with a one-line summary at the top. Record corrections and
confirmed approaches alike, including why they mattered. Don't save what the repo or
chat history already records; update existing notes rather than duplicating; delete
notes that turn out wrong.
```

**S9 — Same design every time / house-style lock-in** *(Opus 4.8 frontend)*
```text
Before building, propose 4 distinct visual directions tailored to this brief (each as:
bg hex / accent hex / typeface — one-line rationale). Ask me to pick one, then implement
only that direction. Avoid generic-AI aesthetics: overused fonts (Inter, Roboto, system
fonts), purple gradients, cookie-cutter layouts.
```

**S10 — Code review missing findings** *(Opus 4.8 review harnesses)*
```text
Report every issue you find, including ones you are uncertain about or consider
low-severity. Do not filter for importance at this stage — surface everything with a
confidence level and estimated severity so a downstream step can rank them. Coverage
is the goal; a finding filtered out later is better than a real bug silently dropped.
```

**S11 — Dense, unreadable final summaries** *(long agentic sessions)*
```text
Terse shorthand is fine between tool calls; your final summary is for a reader who saw
none of it. Open with the outcome — the sentence they'd ask for as the TLDR — then
supporting detail. Complete sentences, no arrow chains or invented labels; when you
mention files, commits, or flags, give each a plain-language clause.
```

**S12 — Thinking too often / too little** *(adaptive thinking, Opus 4.8)*
```text
Thinking adds latency; use it only when it will meaningfully improve answer quality —
typically multi-step reasoning. When in doubt, respond directly.
```
*(Inverse: raise effort first; if pinned low for latency, add "This task involves multi-step reasoning — think carefully before responding.")*
