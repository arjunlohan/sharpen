# Interviewing — finding the unknowns

The skill of prompting *is* the skill of reducing and planning for unknowns. The best prompters aren't the ones with no unknowns — they're the ones who **assume** unknowns and budget moves to surface them before, during, and after the run. This file is the technique layer for step 3–4 of the workflow in [SKILL.md](SKILL.md).

## Scout before you ask a single question

The environment answers most questions faster and more accurately than the user:

- **Read what the prompt touches.** Files named in the request, the module it lives in, the tests around it.
- **Search for prior art.** An earlier implementation, a similar feature, a convention the codebase already settled. "How should errors be shaped?" is usually answered by the three nearest error handlers.
- **Check the history.** Git log and PR descriptions encode decisions the user forgot they made.
- **Search the web** for the library's current API when the prompt depends on it.

Only what survives scouting becomes a question. A user asked something `grep` knows will (rightly) trust the rest of the interview less.

## Ask about the asker

Unknowns live on both sides. Before or alongside the first question round, establish — or infer from context —:

- **Where they are in their thought process** (exploring vs. decided vs. already half-built).
- **Their experience with this domain and codebase** ("I know nothing about the auth modules" changes every question you'd ask).
- **Who the output is for and what it enables.** The single highest-leverage frame: *"I'm working on [larger task] for [who]. They need [what the output enables]. With that in mind: [request]."* Bake this frame into the sharpened prompt itself — models perform better when they understand intent, because context connects the task to relevant information instead of leaving the model to guess.

## The four quadrants, in depth

### Known unknowns → interview

Questions the user knows are open. Technique:

- **Architecture first.** Order by blast radius: data-model and interface questions before UX questions before formatting questions. The canonical script: *"Interview me one question at a time about anything ambiguous — prioritize questions where my answer would change the architecture."*
- **Batch small, default everything.** ≤3–4 questions per round, each with a recommended default so "accept all" is one action. In Claude Code, use the structured question tool (options + a recommended first choice); in plain chat, one question at a time.
- **Two rounds max.** Then decide, write the assumptions into the prompt under an `Assumptions:` block, and proceed. Assumptions written down are cheap to correct; assumptions hidden in the model's head are not.
- **Convert unanswerables into discovery steps.** When the user can't answer ("does the API paginate?"), don't stall — move the question into the prompt: *"Before implementing, check whether the API paginates (see `client/`); if it does, handle cursors; if not, simplify."*

### Unknown knowns → show, don't ask

Taste that can't be articulated but is instantly recognized. Asking produces mush ("something clean?"); showing produces reactions. Moves:

- **Option fans.** *"Make one HTML page with 4 wildly different design directions so I can react to them."* Cheap, parallel, and it breaks the model's own default style. Works beyond design: 10 intervention points ranked cheapest-to-ambitious, 3 API shapes, 4 naming schemes.
- **Throwaway mocks before real wiring.** *"Before wiring anything up, mock the new editor toolbar in a single HTML file with fake data — I want to react to the layout before you touch the real app."* Small spec changes cause drastically different implementations; find the taste **before** the implementation exists.
- **Interactive prototypes with knobs.** For tunables (duration, easing, spacing), a slider beats a debate.

### Unknown unknowns → blind-spot pass

What the user never considered: constraints they don't know exist, historical decisions, what "good" even looks like in an unfamiliar domain. Moves:

- **The literal blind-spot pass.** *"I'm [doing X] but I know nothing about [domain/module]. Do a blind-spot pass: find my unknown unknowns and teach me enough to prompt you better."* Use the literal words "blind spot pass" and "unknown unknowns" — they focus the search.
- **Teach-then-prompt.** When the user can't yet evaluate quality (color grading, transcription accuracy, a new framework): have the model teach the domain's quality criteria *first*, so the user learns what to want. Only then sharpen the task prompt.
- **Surface prior failure modes.** Search issues, incident notes, TODO comments, and reverted commits near the target area; each is an unknown someone already paid for.

### Known knowns → the colleague test

What's stated may still not survive contact. Read the prompt as a colleague with minimal context: is the deliverable named? Is "done" checkable? Are the implicit norms ("obviously we use our design system") actually written down? Anything the user would say "well, obviously—" about goes **in the prompt**, because it isn't obvious to a fresh context.

## References — the highest-bandwidth answer

When a user struggles to describe what they want, stop interviewing and hunt for a reference:

- **Source code is the best reference**, even across languages: *"This Rust crate in `vendor/rate-limiter` implements the exact backoff behavior I want. Read it and reimplement the same semantics in our TypeScript client."* Structure and semantics survive translation; adjectives don't.
- Then, in descending fidelity: a working product to inspect → API docs → diagrams → screenshots → prose.
- Put the pointer in the prompt (`@path`, URL, folder), not a paraphrase of it.

## Question etiquette

- **Never end the turn on a question you could have answered.** Scout, then ask.
- **One decision per question.** Compound questions get half-answers.
- **Make the default the recommendation**, and say why in one clause.
- **Don't re-ask decided things.** If the user already picked X, X is settled unless new evidence appears.
- **Close the loop.** After the interview, reflect the answers back *inside* the sharpened prompt — the user should see their answers became instructions, or they'll stop answering carefully.
