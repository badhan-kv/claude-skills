---
name: explain
disable-model-invocation: true
description: Explain a scientific or technical concept (or how part of the codebase works) at full technical depth, tailored to what the user already knows. First gauges the user's background with a few targeted questions, then gives a rigorous, prerequisite-ordered explanation using proper terminology. Only runs when the user invokes /explain.
tools: Read, Grep, Glob, Bash, AskUserQuestion, WebSearch, WebFetch
---

# Explain

Rigorous, technical explanations calibrated to the user's actual background. Never simplify by
dropping the science; close gaps by supplying the missing prerequisites instead.

`/explain <topic>` — topic may be a concept, a function/file, or a "why" question. With no
argument, take the topic from the preceding conversation.

## Principles

- **Do not dumb down.** Use the correct technical and scientific terms. Elaborate each concept:
  what it is, why it holds, where it stops being valid. A term the user does not know is
  *defined and motivated*, not replaced by a vague synonym.
- **Calibrate with prerequisites, not with less content.** The gauging step finds which
  prerequisites are missing; the explanation supplies those first, in dependency order.
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

### 2. Explain

Order the explanation by dependency, building from what the user has to what the topic needs:

1. **State the question being answered** and the short answer in two sentences (the summary
   the rest justifies).
2. **Define notation once**, up front: every symbol, its units, and what it means physically.
3. **Teach each missing prerequisite** as a numbered section: definition, the mechanism or
   governing equation, the regime of validity, then a worked number (use the project's real
   parameters when available, e.g. compute V for the actual fibre).
4. **Connect to the topic**: derive or walk through the result step by step; show the limit
   cases (what happens when a parameter goes to 0 or ∞) and what would break the assumption.
5. **Map to the code** (if applicable): which line implements which step, and where the code
   approximates or deviates from the theory.
6. **Sources**: textbook/paper/standard reference for each non-obvious result, marked as
   from memory vs looked up. Do not invent citations; if unsure, say so.
7. **Check questions** (2-3), numerical or "what happens if", answerable from what was just
   taught. These test understanding and set up the next round.

Format: markdown with numbered sections and short headers; equations in inline code or plain
text since output renders in a terminal; a small table only where it compares things. Length
follows the number of missing prerequisites; do not pad, do not truncate the derivation.

### 3. Iterate

- If the user answers the check questions or says a part is unclear, re-gauge *that part only*
  (one or two targeted questions), then re-explain it from a different angle (different
  analogy, different representation: equation, geometry, numeric example), not a repeat.
- Track which prerequisites are now solid and do not re-teach them.

### 4. Keep the result

- If the project has a reference-doc workflow or `docs/`, offer (once, one line) to record the
  explanation there. Do not write it without being asked.
- If the explanation exposed a gap in the project's docs (missing citation, unstated
  assumption), say so and offer to add it as a backlog item. Do not edit project files
  unprompted, and never mention AI tooling in project files.
