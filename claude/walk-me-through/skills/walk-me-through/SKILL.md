---
name: walk-me-through
description: Teach a senior developer one subject in 3 to 5 short turns (5W2H-oriented, at most 1500 characters each), with a pickable set of teaching frameworks (C4, follow-one-record, Given/When/Then, ADR, jobs-to-be-done, event storming, predict-then-reveal, Bloom, retrieval, Feynman). The agent waits for "continue" between turns and gives a 3-4 question multiple-choice quiz before each new turn. Subjects include architecture, data structures, business rules and caveats, ticket ACs, Epic objectives, a planned implementation, or a PR. Use when the user says "/walk-me-through <subject>", "walk me through X", "teach me X", "help me understand X before I review/implement it". Always use it after a PR review command (for example review-pr) finishes, with the PR and the review findings as the subject.
---

# Walk me through

You teach one subject to a senior developer. The learner knows software engineering well. The learner does not know this codebase, this domain, or this task as well as the code does. Teach the specifics: the decisions, the rules, the data, the trade-offs. Do not teach general framework or language basics unless the learner asks.

Every turn connects back to the **focus**: the concrete thing the learner must do next (review this PR, implement this ticket, debug this flow). Architecture and business context are there to explain the focus, not as separate lectures.

## Phase 0 — Ground the subject (silent, before any teaching)

Read before you teach. Every claim in a turn must come from a source you read in this session.

1. Find the focus and its sources:
   - PR: `gh pr view <n>` (body and PR-level comments), the diff (by file, in small parts), the linked ticket.
   - Ticket: the issue, its Epic, linked tickets and PRs (use the issue-tracker tools when they are connected).
   - Code area: the repo docs or wiki first (for example `.claude/wiki/`, `docs/`), then the source files.
2. Trace the real flow end to end: entry point → services → data written/read → external calls → output.
3. Collect repo-relative `file_path:line` references for every point you plan to teach.
4. If a fact cannot be verified (for example, production data volume), mark it as unverified in the turn. Do not guess.

Use a search subagent for broad searches. Keep the conclusions; do not paste file dumps into the chat.

## Phase 1 — Intake (one AskUserQuestion call)

Ask only what the context does not already answer. At most 4 questions:

- **Goal:** review / implement / debug / general understanding.
- **Familiarity** with this area: new / used it / changed it before.
- **Focus areas** (multiSelect): architecture fit, data structures and flow, business rules and caveats, ticket ACs and Epic objectives.
- **Depth:** 3 turns (overview) / 4 / 5 turns (deep).

Then show the plan in chat:
- The turn titles (3 to 5), one line each, with the framework of each turn in brackets. Example: `Turn 2 — Where it fits [C4]`.
- The mechanics in use (see "Mechanics frameworks").
- A short "Not covered here" list.
- One line: "Reply with a swap (for example `turn 3: event storming`) at any time."

Start Turn 1 in the same message. If the learner asks for a swap, apply it from the next turn on and update the recap file.

## Framework toolbox

Pick frameworks from this toolbox for each session. The learner can ask for any of them by name, at intake or at any time. Each framework has a "use when" rule. Do not use a framework only because it is on the list.

### Turn-shape frameworks (one main framework per turn)

| Framework | Use when | What the turn contains |
| --- | --- | --- |
| **C4 model** | The learner must place a change in the system: which service, which component, which boundary. | Zoom from context → container → component → code. Stop at the level the focus needs. |
| **Follow one record** | The subject involves data structures, a pipeline, sync, or state changes. | One real (or realistic) record from input to storage to output. Show the fields that matter at each step. |
| **Given/When/Then** | The subject has ACs, business rules, or edge cases that must be exact. | Each rule or AC as a scenario. Include the caveat scenarios, not only the happy path. |
| **ADR (context / decision / consequences)** | The learner asks "why is it built this way", or the code has a non-obvious design. | The forces at the time, the option taken, the options rejected, the cost paid now. |
| **Jobs-to-be-done** | The subject is an Epic, a feature, or a ticket whose business reason is unclear. | The user, the job, the current pain, the success signal. Map ACs to the job. |
| **Event storming** | The subject is a business process with many actors, steps, or async events (webhooks, jobs, external systems). | Events in time order, the command and actor behind each, the policies that react. |
| **Predict-then-reveal** | The behavior is surprising, or the learner has strong prior knowledge that can mislead. | Ask for a prediction first (in the previous turn's **Predict** line), then show the real behavior and why. |

5W2H is the base layout of every turn (see "Turn format"). The turn-shape framework decides the order and the content inside that layout.

### Default mapping

Use this mapping when the learner does not ask for something else:

| Turn | Default framework | Alternative |
| --- | --- | --- |
| Why it exists | Jobs-to-be-done | Given/When/Then for AC-heavy tickets |
| Where it fits | C4 model | Event storming for async or multi-actor flows |
| Data and flow | Follow one record | Event storming for event-driven flows |
| Rules and caveats | Given/When/Then | ADR when the caveats come from past decisions |
| The focus | Predict-then-reveal | ADR for design reviews; Given/When/Then for implementation |

### Mechanics frameworks (they apply across all turns)

| Framework | Default | Use for |
| --- | --- | --- |
| **Bloom's taxonomy** | On | Quiz difficulty rises by turn: understand → apply → analyze → evaluate. |
| **Retrieval + interleaving** | On | From the Turn 2 quiz onward, one question covers an earlier turn. |
| **Feynman check** | Offered at wrap-up | The learner explains the focus in 2-3 sentences. The gaps feed the follow-up prompts. |

The learner can turn a mechanic off (for example "no retrieval questions"). Bloom and retrieval stay on unless the learner asks.

## Turn order

Order the turns from the outside in, then end on the focus:

1. **Why it exists:** business problem, users, Epic objective, ticket ACs.
2. **Where it fits:** system context, the components involved, which service owns what, external systems.
3. **Data and flow:** the data structures, tables or resources, and the fields that matter.
4. **Rules and caveats:** business rules, edge cases, failure modes, past decisions and their cost.
5. **The focus itself:** the PR, the planned change, or the bug, mapped onto the turns before it.

Merge or drop turns to fit the chosen depth. The focus turn is never dropped.

## Turn format

Each turn is **at most 1500 characters** (about 220 words), code references included. If the draft is longer, cut content, not clarity. Use the 5W2H items as a loose guide. Use only the items that carry real information in this turn:

- **What** — the thing, in one or two sentences.
- **Why** — the business or technical reason.
- **Who** — users, services, teams, external systems involved.
- **Where** — `file_path:line` references (repo-relative). 2 to 4 per turn.
- **When** — trigger, timing, order of events, job schedule.
- **How** — the mechanism, step by step.
- **How much** — volume, cost, risk, blast radius. Mark unverified numbers.

Layout:

```
## Turn N/M — <title> [<framework>]
<content, shaped by the framework, short bullets or short paragraphs>
**Link to focus:** <one sentence: how this turn explains the PR / task>
**Predict:** <optional: one question the learner should think about before the next turn>
Reply `continue`, or ask a question.
```

Rules:
- One idea per bullet. No metaphors. No filler intros or summaries.
- Show code only when a 1-5 line excerpt says more than a sentence.
- Teach decisions and trade-offs, not a list of file contents.

## Between turns

- **Stop and wait** after each turn. Never start the quiz or the next turn without the learner's message.
- If the learner asks a question: answer it in at most 800 characters, with file references. Note it in the recap file. Then wait again for `continue`.
- If the question is outside the plan and needs more than a short answer, give a suggested prompt for another chat (see Wrap-up) and continue the plan.

## Quiz (on every `continue`, before the next turn)

One AskUserQuestion call with 3 or 4 questions about the turn just taught.

- Each question has 3 or 4 options. Exactly one is correct.
- **Shuffle:** put the correct option at a different position in each question. Across one quiz, do not use the same position for all questions. Never add "(Recommended)" or any hint to an option.
- **Distractors** are plausible: real misconceptions, the behavior of a similar code path, the rule before a change. All options have about the same length and form.
- **Bloom:** Turn 1 quiz tests understanding. Later quizzes test application ("given this record, what happens?") and analysis ("which change breaks X?").
- **Retrieval:** from the Turn 2 quiz onward, one question covers an earlier turn, preferably one with a wrong answer.
- Match the question style to the turn's framework. Examples: a Given/When/Then turn asks for the "Then" of a new "Given"; a C4 turn asks which component owns a responsibility; a follow-one-record turn asks for a field value at a later step.
- Do not ask for trivia (line numbers, exact method names) unless the name itself carries meaning.
- `header` is short, for example `Q1 rules`.

After the answers:
- Show the score, for example `3/4`.
- For each wrong answer: the correct answer, why, and one `file_path:line` to verify it. At most 300 characters per correction.
- Add each gap to the recap file. Use the gaps in the next turn and in a later retrieval question.
- Then give the next turn in the same message.

After the last turn: give its quiz, then the wrap-up.

## Recap file

Path: `_scraps/tasks/<conversation-theme>/walkthrough.md` in the repo's main folder, unless the user's instructions name another notes folder. Make sure the folder is not tracked by git. Outside a repo, use the session scratchpad.

Content, updated after each quiz:
- Subject, focus, plan, and the framework of each turn.
- Per turn: 3-5 key points with file references.
- Quiz results per turn and the open gaps.
- Learner questions and short answers.

## Wrap-up (after the last quiz)

1. Score per turn, and the gaps that are still open.
2. **Feynman check (optional):** ask the learner to explain the focus in 2-3 sentences. Correct gaps in at most 500 characters.
3. **Prompts for other chats:** 2-4 copy-paste prompts in code blocks, for topics this session did not cover or gaps that need more depth. Each prompt is self-contained: subject, why it matters for the focus, repo, key files, a suggested framework, and "use /walk-me-through" where it fits.
4. The recap file path.

## Review mode (after a PR review command)

Input: the PR and the review findings that the review command produced in this conversation.

1. **Verify the review first.** Before Turn 1, check each finding again against the diff and the code. Classify each one: confirmed / wrong / needs data. Tell the learner of any finding you corrected or dropped, and why. The review must be accurate before it becomes teaching material.
2. Plan the turns with the standard order. The last turn (or two) teaches the findings: what each finding protects against, its severity, and how to verify it. Default framework for the findings turn: predict-then-reveal (the learner judges each finding before the explanation).
3. Quiz questions in the findings turn test whether the learner can judge a finding ("is this a real bug? which input triggers it?"), not only recall it.
4. Do not post anything to GitHub. Comment drafts stay in the chat for the learner's approval.
