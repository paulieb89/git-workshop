# Lesson 3 — PR: Draft → Review → Merge

**A PR is a conversation, not just a button.**

---

## The scenario

Your `fix/show-empty-crash` branch is pushed. Now open a Pull Request so someone can review it before it goes into `main`.

---

## Open a draft PR (GitHub CLI)

```bash
# Draft = work in progress, not ready to merge yet
# Use this when you want early feedback or want to signal "I'm on this"
gh pr create \
  --title "fix: show command crash on empty counters" \
  --body "Fixes crash when tally show is run with no counters. Added a guard and a test." \
  --draft
```

A draft PR shows up in the repo but can't be merged yet. It tells your team: *this exists, don't duplicate it.*

---

## When it's ready, mark it for review

```bash
# Mark it ready
gh pr ready

# Or on GitHub: click "Ready for review" on the PR page
```

---

## The review

Your reviewer will:
- Leave inline comments on specific lines
- Request changes or approve

If they request changes:
```bash
# Make the fix on the same branch
git add tally/cli.py
git commit -m "address review: handle None case too"
git push origin fix/show-empty-crash
# The PR updates automatically
```

---

## Merge it

Once approved:
```bash
# Squash merge keeps main history clean — one commit per feature
gh pr merge --squash --delete-branch
```

Or merge on the GitHub UI. `--delete-branch` tidies up the remote branch automatically.

---

## Key insight

> Draft PRs mean nothing is a surprise. Reviewers see work-in-progress early — they catch the wrong direction before you've gone too far.

---

**Next:** Lesson 4 — letting a robot catch your mistakes before your reviewer has to.
