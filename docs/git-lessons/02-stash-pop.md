# Lesson 2 — Stash and Pop

**Save your place without committing half-done work.**

---

## The scenario

You're mid-way through the `reset` command. It doesn't work yet. Then a bug lands — `tally show` crashes when there are no counters, and someone needs it fixed now.

You can't commit what you've got (it's broken). You don't want to lose it. That's what `stash` is for.

---

## Commands

```bash
# You're on feat/reset-command, mid-edit
git status
# → modified: tally/cli.py

# Stash your work-in-progress
git stash push -m "half-done reset command"

# Now your working tree is clean — create a hotfix branch
git checkout main
git checkout -b fix/show-empty-crash

# Fix the bug, commit it
git add tally/cli.py
git commit -m "fix show command crash when no counters exist"
git push origin fix/show-empty-crash

# Open a PR, get it merged (that's Lesson 3)
# Once it's merged, come back here

# Return to your feature branch
git checkout feat/reset-command

# Retrieve your stashed work
git stash pop

# You're exactly where you left off
git status
```

---

## Stash is a stack

You can stash multiple times. `git stash list` shows everything saved.
`git stash pop` takes the most recent. `git stash pop stash@{1}` takes a specific one.

Don't leave things in the stash for days — it becomes a graveyard. Stash is a short-term pause, not storage.

---

## Key insight

> Stash is a bookmark. Use it to switch context cleanly, then come straight back.

---

**Next:** Lesson 3 — turning that hotfix branch into a PR and getting it merged.
