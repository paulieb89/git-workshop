# Facilitator Guide — Git Lessons with Claude Code

**For:** Learners who know `git commit` but not much else, working alongside a facilitator.  
**How it runs:** You explain the concept. The learner tells Claude Code what to do. You review together.

The goal is not to memorise commands. It's to understand the vocabulary and the *why* well enough to direct an AI agent — and to review what it produces.

---

## The format, explained to the learner

> "You won't be typing git commands. Claude Code will do that. What you need is to understand what you're asking for well enough to say it clearly, and to recognise whether what came back is right. By the end of this, you'll be able to run the same workflow an agent runs — because you'll know exactly what each step is for."

---

## Lesson 1 — Branches

**Concept to explain**

A branch is an isolated copy of the codebase where you can make changes without affecting anyone else's work. It costs nothing to create. The rule is simple: one idea, one branch. Main stays clean.

**Vocabulary**

- `branch` — a parallel line of work
- `main` — the authoritative version of the code, should always be stable
- `commit` — saving a snapshot of your changes with a message
- `push` — sending your commits to GitHub

**What to say to Claude Code**

> "Create a branch called `feat/reset-command`. On that branch, add a reset command to `tally/cli.py`. Commit it with a sensible message, then push the branch."

Or if they're using their own repo:

> "Create a new branch for [the thing you're building]. Make a small change on it, commit it, and push it."

**What to look for**

- Claude should create a branch with a descriptive name (not `fix-thing` — something like `feat/reset-command`)
- The commit message should describe *why*, not just *what*
- `main` should be unchanged — verify with "show me what's on main"

**Discussion**

- "What would have happened if we'd made this change directly on main?"
- "When would you create a branch? What's the trigger?"

---

## Lesson 2 — Stash

**Concept to explain**

You're mid-way through something when an urgent bug lands. You can't commit what you've got — it's broken. Stash lets you set aside unfinished work, fix the urgent thing, then come back to exactly where you were.

Think of it as a bookmark. It's short-term — not somewhere to leave things for days.

**Vocabulary**

- `stash` — temporary shelf for work-in-progress
- `stash pop` — retrieve the most recent thing you shelved
- `working tree` — the files as they currently are on disk (as opposed to what's been committed)

**What to say to Claude Code**

> "I'm mid-way through something on `feat/reset-command` and there's an urgent bug to fix. Stash my current changes, create a new branch called `fix/show-crash` from main, make a small fix there, commit it, and push it. Then come back to the feature branch and restore my stashed work."

**What to look for**

- Claude should stash before switching branches — if it doesn't, ask why the order matters
- After popping, `git status` should show the same uncommitted changes that were there before

**Discussion**

- "What would have happened if you'd just committed the half-done work before switching?"
- "When would stash not be the right tool?" (answer: when it'll be more than a day or two — better to commit as WIP on the branch)

---

## Lesson 3 — Pull Requests

**Concept to explain**

A PR is a formal request to merge one branch into another. Before anything lands on main, it goes through a PR — which means someone reviews it, and CI runs on it.

A **draft PR** signals "I'm working on this" without being ready to merge. It's useful for early feedback and for making sure two people don't work on the same thing.

**Vocabulary**

- `PR` (Pull Request) — a request to merge changes, with a built-in review step
- `draft PR` — a PR that's explicitly not ready to merge yet
- `squash merge` — collapsing all commits from a branch into one before merging, keeping main history clean
- `reviewer` — the person who reads the diff and approves or requests changes

**What to say to Claude Code**

> "Open a draft PR for the `fix/show-crash` branch. Title it 'fix: show command crash on empty counters'. Give it a one-sentence description of what the fix does."

When ready to merge:

> "Mark the PR as ready for review, then merge it with a squash merge and delete the branch."

**What to look for**

- Draft PR should have a clear title and a useful description — not just "changes"
- After merge, the branch should be deleted
- The squash merge means one clean commit appears on main, not every WIP commit from the branch

**Discussion**

- "Why is a description important on a PR?" (context for the reviewer, and for you six months later)
- "What's the difference between a draft PR and just... not opening one yet?"

---

## Lesson 4 — CI

**Concept to explain**

CI (Continuous Integration) is an automated process that runs every time code is pushed to a PR. It runs your tests and any other checks. If they fail, the PR is blocked from merging.

This means you can set main to be physically unbreakable — no code can enter it unless the robot gave it a green tick.

**Vocabulary**

- `CI` — automated checks that run on every PR (tests, linters, etc.)
- `GitHub Actions` — GitHub's built-in system for running CI
- `branch protection` — a GitHub setting that blocks merges unless CI passes
- `red / green` — failed / passed CI (from the coloured status icons on a PR)

**What to say to Claude Code**

> "Create a branch called `feat/rename-command`, add a rename command to `tally/cli.py`, but introduce a deliberate small bug so CI will fail. Open a draft PR, then push it."

Then, once CI has gone red:

> "Fix the bug on the same branch and push again. I want to watch CI go green."

**What to look for**

- After the first push, the PR should show a red CI status
- After the fix, it should go green without opening a new PR (Claude should push to the same branch)
- Walk through the CI output together — identify which test failed and why

**Discussion**

- "If CI didn't exist, whose job would it be to catch that bug?"
- "Why does CI only work *because* we're using branches?"

**Bonus (if time):** Have Claude set up branch protection on the repo. "Enable branch protection on main — require CI to pass before anything can be merged."

---

## Lesson 5 — Releases

**Concept to explain**

A release is a deliberate, named moment in your project's history. It involves three things: bumping the version number, publishing the package (to PyPI, or deploying to a server), and tagging that exact commit so you can always return to it.

This all happens on a `release/` branch — not on main — so if something goes wrong during the release process, main keeps moving and the release branch stays frozen.

**Vocabulary**

- `release branch` — a dedicated branch for cutting a release, keeps main stable during the process
- `version bump` — updating the version number in the project config
- `tag` — a permanent named bookmark on a commit in git history (`v1.0.0`, `v2.3.1`)
- `GitHub Release` — a GitHub page that packages the tag with release notes and downloadable files
- `PyPI` — the Python package registry (`pip install something` comes from here)

**What to say to Claude Code**

> "Create a release branch called `release/1.0.0`. Bump the version in `pyproject.toml` to `1.0.0`. Commit it. Open a PR titled 'release: 1.0.0'. Don't publish to PyPI yet — I'll do that after the PR is merged."

After merge:

> "Tag `main` at the current commit as `v1.0.0` and push the tag. Then create a GitHub Release for `v1.0.0` with a short release note."

**What to look for**

- The version bump commit should be clean and clearly labelled
- After tagging, `git log --oneline` should show the tag next to the commit
- The GitHub Release should be visible on the repo's Releases page

**Discussion**

- "Why not just push directly to main and tag it there?"
- "What does it mean practically to be able to `git checkout v0.9.0`?"

---

## Lesson 6 — Agents

**Concept to explain**

Everything in Lessons 1–5 was done by you, with Claude executing. In Lesson 6, Claude does all of it autonomously — and it uses the same workflow. Branches, PRs, CI, tags. The difference is you're reviewing what an agent produced, not what a colleague produced.

The branch is the circuit breaker. An agent writing directly to main would be terrifying. An agent opening a PR is just another PR to review.

**Vocabulary**

- `claude -p` — non-interactive Claude Code, takes a single prompt and runs to completion
- `agentic PR` — a PR opened by an AI agent rather than a human
- `review gate` — the mandatory human checkpoint between "agent wrote this" and "this ships"

**What to say to Claude Code**

Single-shot feature:

> "Add a 'decrement' command to `tally/cli.py` that reduces a counter by 1. Follow the existing pattern. Add a test. Create a branch called `feat/decrement`, commit the changes, and open a draft PR."

Release automation:

> "Cut release 1.1.0. Create a release branch, bump the version in `pyproject.toml`, run the tests, and open a PR titled 'release: 1.1.0'. Do not publish to PyPI — I'll do that after the PR is merged."

**What to look for**

- The agent should produce a reviewable diff — check it like you would any PR
- CI should still run and pass
- Nothing should land on main until you explicitly merge

**Discussion**

- "What would have gone wrong if this repo didn't use branches?"
- "What's your job when reviewing an agentic PR — what are you actually checking?"
- "At what point does this workflow stop feeling manual and start feeling like control?"

---

## Closing frame

> "The thing that makes agents safe to use is the same thing that makes your own work safe to review: branches and PRs. You haven't learned a set of commands. You've learned the workflow that every good engineer and every good agent uses. The commands are just how you talk to git — you can always look those up, or let Claude handle them. Understanding *why* the workflow exists is what makes you able to direct it."
