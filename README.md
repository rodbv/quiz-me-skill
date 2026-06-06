# quiz-me

A Claude Code skill that quizzes you on code changes, specs, or plans to verify genuine understanding — especially useful when working with AI-generated code.

## What it does

Instead of explaining code to you, it reads an artifact and asks you Socratic questions to test what you actually know. Forces active recall before you commit, push, or start building.

**Three modes:**
- **Code mode** (default) — quizzes on staged/unstaged git changes before a commit
- **Spec mode** — quizzes on a design doc before you implement it
- **Plan mode** — quizzes on an implementation plan and architectural decisions

## Install

```bash
npx skills add rodbv/quiz-me-skill
```

## Usage

```
/quiz-me                          # quiz on current staged+unstaged changes
/quiz-me last commit              # quiz on the previous commit
/quiz-me on the spec              # quiz on docs/superpowers/specs/ (latest file)
/quiz-me on the plan              # quiz on docs/superpowers/plans/ (latest file)
/quiz-me <path/to/file.md>        # quiz on a specific spec or plan file
```

## Example

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

## Why

When using AI agents to write code, it's easy to accept changes you don't fully understand. This skill creates a forcing function: you can't just rubber-stamp a diff. It asks you *why* the code works, what breaks if you remove a line, what a migration does to existing data. If you can't answer, you learn before it ships.

Works well alongside `grill-me` (which stress-tests your design decisions) and before commits or PRs.

## Credits

Built by [@rodbv](https://github.com/rodbv). Inspired by the problem of vibe-coding without understanding.
