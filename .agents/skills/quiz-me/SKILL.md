---
name: quiz-me
description: Quiz the user on code changes, a spec, or an implementation plan to verify genuine understanding before committing, building, or approving. Especially valuable when working with AI-generated code or spec-driven development. Use when the user says "quiz me", "/quiz-me", "test my understanding", "quiz me before I commit", "quiz me on this spec", "quiz me on the plan", "make sure I understand this", or anything suggesting they want to verify comprehension of a diff, design doc, or architectural plan.
---

# Quiz Me

Your job: read an artifact (code diff, spec, or plan) and quiz the user to verify genuine understanding - not explain it to them. This is active recall, not passive review. The goal is to surface gaps before they become bugs or wrong implementations.

## Step 1: Determine source

Three modes, detected from context or user request:

**Code mode (default):** Read git changes.

- Default: `git diff HEAD` (staged + unstaged)
- "last commit" / "previous commit" → `git diff HEAD~1 HEAD`
- A specific SHA → `git diff <sha>~1 <sha>`
- "last N commits" → `git diff HEAD~N HEAD`
- A PR number → use gh MCP/plugin

**Spec mode:** Triggered when the user mentions a spec, design doc, or "before I implement".

- Read the specified file path, or find the most recent file in `docs/superpowers/specs/`
- Questions target: data model decisions, interface contracts, scope boundaries, constraints

**Plan mode:** Triggered when the user mentions a plan, implementation plan, or "architectural decisions".

- Read the specified plan file, or look for recent plan files in `docs/superpowers/plans/`
- Questions target: _why_ this approach over alternatives, ordering of steps, dependencies between phases, risks

If ambiguous, ask the user which artifact to quiz on before proceeding.

## Step 2: Generate 5–7 questions

Read the artifact carefully. Generate questions that require _thinking_, not just reading the artifact back. A good question cannot be answered by copy-pasting a line from the source.

**For code diffs:**

- Why was this approach chosen over a simpler/different one?
- What breaks if this line/block is removed?
- What happens with null, empty, or unexpected inputs?
- What does this migration change for existing data?
- What other parts of the system does this affect?

**For specs:**

- Why is [design decision X] made this way rather than [obvious alternative]?
- What's the contract of the provider protocol - what must every implementation guarantee?
- What's out of scope, and why?
- What happens if [constraint from spec] is violated?
- How does [component A] communicate with [component B]?

**For plans:**

- Why does Phase 1 end here rather than including [next thing]?
- Why was [architectural decision] chosen over [alternative you spotted]?
- What's the dependency between [step A] and [step B] - could they be reversed?
- What's the riskiest assumption in this plan?
- If [external dependency] isn't available, what breaks first?

Scale to what's in the artifact - a two-line fix warrants different depth than a full system design.

## Step 3: Run the quiz

1. Ask one question
2. Wait for the answer
3. Grade it - see Grading below
4. Move to the next
5. Final summary: X/Y + one observation

## Grading

Correct = understands the _why_, not just the _what_. Partial credit when the right idea is there but an important nuance is missing. Don't accept vague answers - if they say "it's more efficient", ask them to be specific.

**On partial or wrong answers:** ask one follow-up question that exposes the gap rather than explaining it - let them reason to the answer. Only reveal the correction if they miss the follow-up too.

Good follow-up patterns:

- "You said X - what happens when [edge case Y]? Does your answer still hold?"
- "If that were true, what would [specific scenario] do?"
- "What does the spec say about [the thing they glossed over]?"

If they still miss it after the follow-up, give a short correction and move on. Not a lecture.

## Tone

Tough but supportive senior dev. Not a gotcha machine - you want them to succeed. If they ace it, say so clearly.
