# Lesson 5 — Release: PyPI, Server Deploy, GitHub Release

**Ship it deliberately.**

---

## The scenario

`tally` is stable. CI is green. It's time to publish it to PyPI so anyone can `pip install tally`, deploy the latest build to a server, and tag the release in git so you can always come back to this exact point.

All three happen from a `release/` branch, not directly on `main`.

---

## Step 1 — Create a release branch

```bash
git checkout main
git pull origin main
git checkout -b release/1.0.0
```

---

## Step 2 — Bump the version

Edit `pyproject.toml`:
```toml
[project]
version = "1.0.0"
```

Commit it:
```bash
git add pyproject.toml
git commit -m "chore: bump version to 1.0.0"
```

---

## Step 3 — Publish to PyPI

```bash
# Build the distribution
python -m build

# Upload to PyPI (you'll need a PyPI API token)
python -m twine upload dist/*
```

Or with uv:
```bash
uv build
uv publish
```

Anyone can now run `pip install tally`. Test it:
```bash
pip install tally
tally show
```

---

## Step 4 — Deploy to a server

If you're running `tally` as a service (e.g. a web API wrapper):

```bash
# SSH to your server and pull the latest
ssh user@yourserver.com

cd /srv/tally
git pull origin main          # or copy the wheel via scp
pip install --upgrade tally
sudo systemctl restart tally  # if running as a systemd service
```

For a simpler deploy via the wheel:
```bash
scp dist/tally-1.0.0-py3-none-any.whl user@yourserver.com:/tmp/
ssh user@yourserver.com "pip install /tmp/tally-1.0.0-py3-none-any.whl --force-reinstall"
```

---

## Step 5 — Merge the release branch and tag it

```bash
# PR the release branch into main
gh pr create \
  --title "release: 1.0.0" \
  --body "Bumps version to 1.0.0. Published to PyPI."

# After merge, tag main at this point
git checkout main
git pull origin main
git tag -a v1.0.0 -m "Release 1.0.0"
git push origin v1.0.0
```

---

## Step 6 — Cut the GitHub Release

```bash
gh release create v1.0.0 \
  --title "tally 1.0.0" \
  --notes "First stable release. Adds reset and rename commands." \
  dist/tally-1.0.0-py3-none-any.whl
```

The wheel is attached. Anyone can download it directly from GitHub without PyPI.

---

## Why a release branch?

If a last-minute bug is found during release prep, you fix it on `release/1.0.0` — not on `main`. Main keeps moving. The release branch stays frozen until it ships.

---

## Key insight

> A tag is a permanent bookmark in your history. `git checkout v1.0.0` takes you back to exactly this moment — even two years from now. That's what makes rollbacks possible.

---

**Next:** Lesson 6 — doing all of this with an agent.
