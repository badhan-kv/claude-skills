---
name: explain
disable-model-invocation: true
description: Explain a scientific or technical concept (or how part of the codebase works) at full technical depth, tailored to what the user already knows. First gauges the user's background with a few targeted questions, then teaches one small step at a time as a conversation, proposing the next step after each one. Only runs when the user invokes /explain.
tools: Read, Grep, Glob, Bash, AskUserQuestion, WebSearch, WebFetch
---

# Explain

Rigorous, technical explanations calibrated to the user's actual background, delivered as a
conversation: one small step per message, never one long dump. Never simplify by dropping the
science; close gaps by supplying the missing prerequisites, one at a time.

`/explain <topic>` — topic may be a concept, a function/file, or a "why" question. With no
argument, take the topic from the preceding conversation.

## Principles

- **Do not dumb down.** Use the correct technical and scientific terms. Elaborate each concept:
  what it is, why it holds, where it stops being valid. A term the user does not know is
  *defined and motivated*, not replaced by a vague synonym.
- **No non-technical or non-scientific analogies or wording.** Explain with equations, geometry,
  limit cases, worked numbers, diagrams in text, or code, not everyday metaphors.
- **One step per message.** Each message teaches one idea. A chunk is small: roughly 5-10 lines,
  one concept, at most one equation or one worked number. The depth comes from the sequence of
  steps, not from the size of any one of them. If a step feels like it needs more room, it is two
  steps.
- **Never run ahead.** After a step, stop and wait for the user. Do not continue to the next
  step, and do not pre-empt questions by adding "while we're here" material.
- **Every message is self-contained.** Restate what the reader needs to follow it: what each
  symbol means (with units), what each term means, what was concluded in the previous step.
  This applies most to re-explanations. Do not make the user scroll up or remember a
  definition, even if it was covered before. Keep the restatement to a clause or a line, not a
  recap section.
- **Calibrate with prerequisites, not with less content.** The gauging step finds which
  prerequisites are missing; they are taught first, in dependency order, one per step.
- **Ground it.** When the topic is in the codebase, read the actual code (cite
  `file_path:line_number`) and explain what it does, not what such code usually does. For
  general science, look up the standard source first (textbook, original paper, docs) and name it.
- **Say what is verified.** Tag claims: *verified* (derived or checked here, show how) vs
  *taken from the code/docstring/source, not independently checked*. Re-derive small results
  (normalisation constants, limits, orders of magnitude) instead of asserting them, and plug in
  the project's real numbers when they exist.
- **No blame, no relitigating.** If the user does not understand, the explanation was
  insufficient. Never imply they should already know it.

## Procedure

### 1. Gauge understanding (first, before explaining)

Skip only if the user has already stated their background for this topic, or a saved memory
records it (check the memory index for an entry on this topic or domain).

1. Identify the 3-6 prerequisite ideas the explanation will rest on (e.g. for fibre coupling:
   ray vs wave optics, guiding mechanism, NA / V-number, Gaussian beams, overlap integral).
   Read the relevant code/docs first so the list matches what the answer will need.
2. Ask with `AskUserQuestion` (max 4 questions per call; split across calls if needed). Each
   question probes one prerequisite using a *concrete* question, not "do you know X?":
   - Offer 2-4 options that are specific, mutually exclusive, and include plausible
     misconceptions, so the choice reveals the mental model.
   - Put the correct/expected option in a neutral position, not always first, and never label it.
   - Give each option a one-line description in neutral terms.
   - Include a "Not sure" style option where it fits; "I don't know" is a valid answer.
3. Read the answers as a map of gaps. A wrong answer or "not sure" marks a prerequisite to teach;
   a correct answer marks one to state briefly and move past.
4. Turn the map into an ordered **queue of steps** (missing prerequisites in dependency order,
   then the topic itself). Keep it in mind; do not print the whole queue as a lecture.

### 2. Teach, one step at a time

Start with the question being answered and the short answer in two sentences, then name the
first step. After that, each message is one step from the queue:

- **Step content** (as the idea needs): definition, the mechanism or governing equation, the
  regime of validity, a worked number (use the project's real parameters when available). Define
  every symbol used, in this message, with units. For steps that connect to the topic: derive or
  walk through the result, show limit cases, say what would break the assumption. For steps that
  map to code: which line implements which part, and where the code approximates or deviates
  from the theory. Cite a source inline for a non-obvious result, marked as from memory vs
  looked up; do not invent citations.
- **End with a proposal, not a question.** The last line names the next step so the user can
  confirm in a word or two, e.g. "Next: the V-number and what it says about the number of guided
  modes. Go on?" or "Next: how the code computes this (`coupling.py:88`)." Do not quiz the user
  at the end of a step. If they are not satisfied they will ask; that is the signal.
- Then **stop** and wait.

Format: short markdown, equations in inline code or plain text since output renders in a
terminal; a small table only where it compares things. No long section structure, no numbered
multi-part answers.

### 3. Handle the reply

- **Confirmation** ("ok", "yes", "go on"): acknowledge in a few words at most and give the next
  step.
- **Confusion or a question about the step**: answer that point only, at chunk size. For a
  re-explanation, use a different technical angle (equation, geometry, limit case, numeric
  example), not a repeat of the same wording and not a lay analogy, and restate the definitions
  it relies on (symbols, terms, the previous conclusion) even if they were covered earlier.
  Then propose resuming.
- **Question about a term or symbol in the last chunk**: define it fully and briefly, then
  propose continuing.
- **Tangent**: answer it briefly, at chunk size, then ask whether to return, naming the exact
  point left open, e.g. "Back to the overlap integral where we paused, or stay on this?"
- **Several questions at once**: answer them one at a time, in order, each with enough context to
  stand alone; if that makes the reply long, answer the first and say which are queued.
- Track which steps are solid and do not re-teach them, other than the restated context above.
  If the user's answers reveal a gap not in the queue, insert a step for it.

### 4. Keep the result

At the end of the session, or when the user signals they are done:

- If the project has a reference-doc workflow or `docs/`, offer (once, one line) to record the
  explanation there. Do not write it without being asked.
- If the explanation exposed a gap in the project's docs (missing citation, unstated
  assumption), say so and offer to add it as a backlog item. Do not edit project files
  unprompted, and never mention AI tooling in project files.
