# Lesson 1 — Branch, Add, Commit, Push

**One idea. One branch.**

---

## The scenario

You've forked `tally` — a tiny CLI for counting things. You want to add a new `reset` command.

Don't touch `main`. Create a branch instead.

---

## Commands

```bash
# See where you are
git status
git branch

# Create a branch and switch to it
git checkout -b feat/reset-command

# Make your change — open tally/cli.py and add the reset command
# (see the LESSON.md on this branch for what to add)

# Stage it
git add tally/cli.py

# Commit it — write why, not what
git commit -m "add reset command to clear a counter"

# Push to YOUR fork
git push origin feat/reset-command
```

---

## Why not just commit to main?

Right now you're the only one working here, so it feels fine.

But imagine:
- A colleague pushes a bug fix to `main` while you're halfway through your feature
- You have to resolve the conflict mid-feature
- Or worse — you push your half-done feature over their fix

A branch keeps your work **isolated until it's ready**. Main stays clean.

---

## Key insight

> A branch costs you 10 seconds. A merge conflict on main costs you 10 minutes and a bad morning.

---

**Next:** Lesson 2 — what to do when you're mid-feature and something urgent lands.
