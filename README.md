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

## Why

When using AI agents to write code, it's easy to accept changes you don't fully understand. This skill creates a forcing function: you can't just rubber-stamp a diff. It asks you *why* the code works, what breaks if you remove a line, what a migration does to existing data. If you can't answer, you learn before it ships.

Works well alongside `grill-me` (which stress-tests your design decisions) and before commits or PRs.

## Credits

Built by [@rodbv](https://github.com/rodbv). Inspired by the problem of vibe-coding without understanding.
