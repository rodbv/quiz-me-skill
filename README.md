# rodbv/skills

Claude Code skills for learning-focused development. Install any skill individually, or all at once.

## Install

```bash
npx skills add rodbv/skills:quiz-me
npx skills add rodbv/skills:guided-coding
```

---

## quiz-me

Quizzes you on code changes, specs, or plans to verify genuine understanding — especially useful when working with AI-generated code.

Instead of explaining code to you, it reads an artifact and asks Socratic questions to test what you actually know. Forces active recall before you commit, push, or start building.

**Three modes:**
- **Code mode** (default) — quizzes on staged/unstaged git changes before a commit
- **Spec mode** — quizzes on a design doc before you implement it
- **Plan mode** — quizzes on an implementation plan and architectural decisions

### Usage

```
/quiz-me                          # quiz on current staged+unstaged changes
/quiz-me last commit              # quiz on the previous commit
/quiz-me on the spec              # quiz on docs/superpowers/specs/ (latest file)
/quiz-me on the plan              # quiz on docs/superpowers/plans/ (latest file)
/quiz-me <path/to/file.md>        # quiz on a specific spec or plan file
```

### Example

**Spec mode** (`/quiz-me on the spec`):

```
> How do you want to be quizzed?
> A) One at a time — I ask a question, you answer, I give feedback, then we move on (recommended)
> B) All at once — I give you all questions upfront, you answer them all, then I grade

a

Question 1 of 6:
ForecastSnapshot stores throughput_snapshot as a JSONField rather than
normalizing it into rows. Why?

> so we know the original dataset used to produce the forecast

Correct. Storing the raw data means the forecast is fully reproducible —
you can re-run it later and get the same result, and you can see exactly
what changed between forecasts (e.g. throughput dropped, a slow item was
added). If you only stored the p50/p85/p95 dates, you'd know the forecast
changed but not why.

Question 2 of 6:
The DataProvider protocol defines fetch_work_items and fetch_deployments.
If someone builds a Jira provider but Jira doesn't expose deployment data
— what should they do, and does the current protocol design allow for it?

> the application should handle missing data. Provider should not crash though

Partially correct. Right — the app should handle missing data gracefully
and the provider shouldn't crash. But the current protocol design doesn't
explicitly document this: fetch_deployments returns list[DeployEvent] with
no empty-list contract stated. A provider with no deployment data should
return [] — but that convention needs to be explicit.

...

Final score: 5.5/6 — strong. You knew the architecture cold.
```

**Code mode** (`/quiz-me before I commit`):

```
Question 1 of 5:
You added db_index=True to the email field. Under what query patterns
does this actually help, and when would it make no difference?

> when filtering or sorting by email, like login lookups

Correct — B-tree indexes speed up equality lookups and range scans.
They don't help with LIKE '%partial%' queries or full-table aggregations.
```

### Why

When using AI agents to write code, it's easy to accept changes you don't fully understand. This skill creates a forcing function: you can't just rubber-stamp a diff. It asks you *why* the code works, what breaks if you remove a line, what a migration does to existing data. If you can't answer, you learn before it ships.

---

## guided-coding

A Socratic coding tutor that guides you through implementing a spec or plan entirely on your own — without writing a single line of code for you.

Works from a spec or implementation plan. Breaks the work into steps, presents one at a time, waits for you to implement, then checks your work by reading your diff and asking probing questions. If you get stuck, it explains concepts and points to documentation — never gives you copy-paste answers.

**Testing modes:**
- **A) TDD** — write the test first, make it pass, then move on
- **B) Test-after** — implement, then add tests before the next step
- **C) Mixed** (default) — TDD for pure logic, skip for glue/config
- **D) No tests** — skip tests entirely

### Usage

```
/guided-coding                    # guide me through the latest spec/plan
/guided-coding on the spec        # guide me through docs/superpowers/specs/ (latest)
/guided-coding on the plan        # guide me through docs/superpowers/plans/ (latest)
/guided-coding <path/to/file.md>  # guide me through a specific file
```

### Example

```
> Before we start — how do you want to handle testing?
> A) TDD — write the test first, make it pass, then we move on
> B) Test-after — implement, then add tests before moving to the next step
> C) Mixed — I'll flag which steps genuinely benefit from TDD
> D) No tests — skip tests entirely

c

I've planned 7 steps. Let's start with Step 1.

─── Step 1: Pydantic models + DataProvider protocol ───

Implement WorkItem, DeployEvent, and DataProvider in kwando_lib/models.py.
WorkItem needs: id (str), started_at and finished_at (datetime | None), item_type (str).
DeployEvent needs: deployed_at (datetime), succeeded (bool).
DataProvider is a Protocol with two methods — both accept `since: datetime`
and return a list. The protocol contract has one rule you'll need to encode
in the docstring, not the type signature.

This step is logic-heavy — write a test for DataProvider first (hint: test
what a provider with no deployment data should return).

Let me know when you're done, or say 'hint' if you get stuck.

> done

[reads git diff]

Looks solid. One question: your fetch_deployments docstring says
"returns empty list if no data" — but what guarantees a third-party
implementer reads that? Is there a way to make the contract more enforceable?

> maybe an abstract base class instead of Protocol?

That's worth thinking through. What do you gain from Protocol vs ABC here?
Which one lets an existing class satisfy the interface without inheriting from it?
```

### Why

The fastest way to understand something is to build it yourself. This skill prevents the temptation of asking Claude to "just write this one thing" — it holds the line so you do the learning. Use it when you have a spec ready and want to implement it as a deliberate practice session.

Pairs well with `quiz-me` (which tests comprehension before you build) and `grill-me` (which stress-tests your design decisions).

---

## Credits

Built by [@rodbv](https://github.com/rodbv). Inspired by the problem of vibe-coding without understanding.
