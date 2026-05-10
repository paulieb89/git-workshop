# Lesson 6 — Agents and Claude Code

**The workflow you just learned is what makes agent work safe.**

---

## The scenario

You've spent five lessons building good habits: branches, PRs, CI, releases. Now an agent can follow the exact same workflow — and because it's using branches, you can **review what it did before it touches main**.

Without branches, an agent writing to main is terrifying. With branches, it's just another PR.

---

## One-shot: add a feature with `claude -p`

```bash
# Ask Claude to add a feature, non-interactively
claude -p "Add a 'decrement' command to tally that reduces a counter by 1. \
Follow the existing pattern in cli.py. Add a test in tests/. \
Create a branch called feat/decrement, commit the changes, and open a draft PR."
```

Claude will:
1. Read the codebase
2. Create the branch
3. Write the code
4. Run the tests
5. Open the PR

You get a PR link. Review it like any other PR. Merge or request changes.

---

## Interactive: pair with Claude Code

```bash
# Start Claude Code in the repo
claude

# In the session, you can say:
# "Create a branch and add an 'export' command that writes all counters to a CSV"
# "Run the tests and fix anything that fails"
# "Open a draft PR when it's ready"
```

Claude Code works in your terminal, reads your files, runs commands, and reports back. You stay in control — every change goes through a branch and a PR, same as if a junior dev wrote it.

---

## Automate the release workflow

```bash
claude -p "We're cutting release 1.1.0. \
Create a release/1.1.0 branch, bump the version in pyproject.toml, \
run the tests, and open a PR titled 'release: 1.1.0'. \
Do not publish to PyPI — I'll do that after the PR is merged."
```

The agent handles the mechanical steps. You handle the judgment calls (approve the PR, trigger the publish).

---

## Why branching makes agents safe

Without branching:
- Agent writes directly to `main`
- Bad code ships immediately
- Rollback is a mess

With branching:
- Agent writes to `feat/decrement`
- CI runs on the PR
- You review the diff
- You decide when it merges

**The branch is the circuit breaker.** It gives you a review gate between "agent generated this" and "this is in production."

---

## Useful `claude -p` patterns

```bash
# Add a feature
claude -p "Add X feature to [file]. Follow existing patterns. Branch, commit, draft PR."

# Fix a failing test
claude -p "The test tests/test_cli.py::test_reset is failing. Fix it on a new branch and open a PR."

# Cut a release
claude -p "Bump version to 1.2.0 in pyproject.toml on a release branch. Open a PR. Don't publish."

# Code review
claude -p "Review the open PR #12 and leave inline comments on anything that looks wrong."
```

---

## Key insight

> Agents are fast and tireless but they make mistakes. Branches and PRs give you a mandatory review gate. The workflow you've spent these lessons building isn't bureaucracy — it's the thing that makes powerful automation safe to use.

---

**You're done.** You now know why branching isn't a faff.
