# Socratic Skills

[🇧🇷 Português](README.pt-br.md)

Claude Code skills for learning-focused development - active recall, guided implementation, and <a href="https://en.wikipedia.org/wiki/Socratic_method" target="_blank">Socratic feedback loops</a>.

## Skills

<table><tr>
<td width="75%">
<p><strong><a href="#quiz-me">quiz-me</a></strong> - Quizzes you on a code diff, spec, or plan one question at a time, probing with a follow-up before correcting a wrong answer</p>
<p><strong><a href="#guide-me">guide-me</a></strong> - Walks you through implementing a spec step by step, checks your work, and gives hints - without ever writing code for you</p>
</td>
<td align="center" width="25%">
<img src="socrates-black.svg" alt="Socrates" width="160" />
</td>
</tr></table>

## Install

```bash
npx skills add rodbv/socratic-skills/skills/quiz-me
```

```bash
npx skills add rodbv/socratic-skills/skills/guide-me
```

---

## quiz-me

Quizzes you on code changes, specs, or plans to verify genuine understanding - especially useful when working with AI-generated code.

Instead of explaining code to you, it reads an artifact and asks questions one at a time to test what you actually know. On partial or wrong answers it probes with a follow-up before correcting - you reason to the answer rather than being told it. Forces active recall before you commit, push, or start building.

**Three artifact types:**
- **Code** (default) - quizzes on staged/unstaged git changes before a commit
- **Spec** - quizzes on a design doc before you implement it
- **Plan** - quizzes on an implementation plan and architectural decisions

### Why

When using AI agents to write code, it's easy to accept changes you don't fully understand. This skill creates a forcing function: you can't just rubber-stamp a diff. It asks you *why* the code works, what breaks if you remove a line, what a migration does to existing data. If you can't answer, you learn before it ships.

### Usage

```
/quiz-me                          # quiz on current staged+unstaged changes
/quiz-me last commit              # quiz on the previous commit
/quiz-me on the spec              # quiz on docs/superpowers/specs/ (latest file)
/quiz-me on the plan              # quiz on docs/superpowers/plans/ (latest file)
/quiz-me <path/to/file.md>        # quiz on a specific spec or plan file
```

### Example

**Spec mode** (`/quiz-me on the spec`) - showing the Socratic follow-up on a partial answer:

```
Question 1 of 5:
The spec stores total_price directly on the Order model rather than
computing it from line items each time. Why?

> so we don't recalculate it on every page load

Performance is part of it. But there's a more important reason -
what happens to a computed total if a product's price changes later?

> the old orders would show the new price instead of what was charged

Exactly. Denormalizing the total isn't just a performance call -
it's a correctness requirement. The price at purchase time is a legal
record. A computed total would silently change as prices update.

Question 2 of 5:
The spec uses a slug field for post URLs instead of a numeric ID. What
breaks if someone renames a post after it's been published?

> the URL changes, old links break

Right. What should the spec say about this - is it addressed?

> ... I don't think it is

That's a real gap. The spec should either forbid renaming after publish,
keep the original slug and add a separate display title, or handle
redirects from old slugs. Worth raising before implementing.

...

Final score: 4/5 - solid. Found one genuine spec gap in Q2.
```

**Code mode** (`/quiz-me` on a diff that added `select_related`):

```
Question 1 of 4:
This diff adds select_related('author') to the post queryset. What
problem was this solving, and how does select_related fix it?

> it was doing a query for every post to get the author, now it's one query

Correct - that's the N+1 problem. select_related does a SQL JOIN so
author data comes back in the same query. When would you use
prefetch_related instead?

> when it's a many-to-many or reverse FK relationship

Exactly. JOIN doesn't work cleanly for those - prefetch_related runs
a second query and does the joining in Python.
```

---

## guide-me

A Socratic coding tutor that guides you through implementing a spec or plan entirely on your own - without writing a single line of code for you.

Works from a spec or implementation plan. Breaks the work into steps, presents one at a time, waits for you to implement, then checks your work by reading your diff and asking probing questions. If you get stuck, it explains concepts and points to documentation - never gives you copy-paste answers.

**Testing modes:**
- **A) TDD** - write the test first, make it pass, then move on
- **B) Test-after** - implement, then add tests before the next step
- **C) Mixed** (default) - TDD for pure logic, skip for glue/config
- **D) No tests** - skip tests entirely

### Why

The fastest way to understand something is to build it yourself. This skill prevents the temptation of asking Claude to "just write this one thing" - it holds the line so you do the learning. Use it when you have a spec ready and want to implement it as a deliberate practice session.

Pairs well with `quiz-me` (active recall before you build) and [`grill-me`](https://github.com/mattpocock/skills/blob/main/skills/productivity/grill-me/SKILL.md) (stress-tests your design decisions).

### Usage

```
/guide-me                    # guide me through the latest spec/plan
/guide-me on the spec        # guide me through docs/superpowers/specs/ (latest)
/guide-me on the plan        # guide me through docs/superpowers/plans/ (latest)
/guide-me <path/to/file.md>  # guide me through a specific file
```

### Example

```
> Before we start - how do you want to handle testing?
> A) TDD - write the test first, make it pass, then we move on
> B) Test-after - implement, then add tests before moving to the next step
> C) Mixed - I'll flag which steps genuinely benefit from TDD
> D) No tests - skip tests entirely

c

I've planned 9 steps. Let's start with Step 1.

─── Step 1: WorkItem model ───

Implement `WorkItem` in kwando_lib/models.py - a Pydantic model with four fields:
id (str), started_at and finished_at (datetime | None), item_type (str).

Let me know when you're done, or say 'hint' if you get stuck.

> done

[reads git diff]

Looks good. One question: both datetime fields are Optional - what happens
if you construct a WorkItem without passing them at all? Is that the intended default?

> yeah, items in progress have no finished_at yet

Makes sense. Move on to Step 2.

─── Step 2: DeployEvent model ───

Add `DeployEvent` to the same file - two fields: deployed_at (datetime), succeeded (bool).

Let me know when you're done.

> done

[reads git diff]

Clean. Step 3 is where the interesting design starts.

─── Step 3: DataProvider protocol ───

Add a `DataProvider` Protocol with two methods, both accepting `since: datetime`
and returning a list. The protocol has one contract rule you can't express in the
type signature - encode it in the docstring.

This step has real logic - write a test for DataProvider first:
what should a provider with no deployment data return?

Let me know when you're done, or say 'hint' if you get stuck.

> done

[reads git diff]

Your fetch_deployments docstring says "returns empty list if no data" - but what
guarantees a third-party implementer reads that? Is there a way to make the contract
more enforceable?

> maybe an abstract base class instead of Protocol?

That's worth thinking through. What do you gain from Protocol vs ABC here?
Which one lets an existing class satisfy the interface without inheriting from it?
```

---

## Credits

Built by [@rodbv](https://github.com/rodbv). Inspired by the problem of vibe-coding without understanding, and by [@mattpocock](https://github.com/mattpocock)'s [grill-me](https://github.com/mattpocock/skills/blob/main/skills/productivity/grill-me/SKILL.md) skill.
