# Git Workflow Lessons — Overview

A practical course for developers who already know `git commit` but skip the rest.

---

## What you'll learn

| # | Lesson | Concepts |
|---|--------|----------|
| 1 | [Branch, Add, Commit, Push](01-branch-add-commit-push.md) | `checkout -b`, `add`, `commit`, `push` |
| 2 | [Stash and Pop](02-stash-pop.md) | `stash`, `stash pop`, context switching |
| 3 | [PR: Draft → Review → Merge](03-pr-draft-merge.md) | draft PRs, review cycle, squash merge |
| 4 | [CI](04-ci.md) | GitHub Actions, branch protection, red/green |
| 5 | [Release: PyPI, Server, Tag](05-release-pypi-deploy.md) | `build`, `publish`, `tag`, `gh release` |
| 6 | [Agents and Claude Code](06-agents-claude-code.md) | `claude -p`, agentic PRs, safe automation |

---

## The vehicle

All lessons use `tally` — a tiny Python CLI for counting things.

```bash
tally add tasks        # increment the 'tasks' counter
tally show             # print all counters
tally reset tasks      # zero a counter
```

It's simple enough to understand in 5 minutes. Complex enough that you'll want branches.

---

## Setup

```bash
# 1. Fork this repo on GitHub (top-right button)

# 2. Clone your fork
git clone https://github.com/YOUR_USERNAME/tally.git
cd tally

# 3. Create and activate a virtual environment

# Mac / Linux
python -m venv .venv
source .venv/bin/activate

# Windows
python -m venv .venv
.venv\Scripts\activate

# 4. Install
pip install -e ".[dev]"

# 5. Check it works
tally add tasks
tally show
pytest
```

> Keep the venv active whenever you use `tally`. You'll see `(.venv)` in your prompt when it is.
> New terminal? Run the `activate` command again before starting.

Start with Lesson 1.

---

## The throughline

Every lesson reinforces the same idea: **branches cost you 10 seconds and save you from an unpredictable amount of pain.** By Lesson 6, an AI agent is opening PRs for you using the exact same workflow — and because you understand branches, you know exactly how to review and control what it does.
