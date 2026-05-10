# git-workshop

A practical Git workflow course and MCP reference library.

## Contents

### Git Lessons

Six hands-on lessons that take you from basic commits to agentic PRs. All lessons use `tally` — a tiny Python CLI — as the practice vehicle.

| # | Lesson | Key concepts |
|---|--------|-------------|
| 1 | [Branch, Add, Commit, Push](docs/git-lessons/01-branch-add-commit-push.md) | `checkout -b`, `add`, `commit`, `push` |
| 2 | [Stash and Pop](docs/git-lessons/02-stash-pop.md) | `stash`, context switching |
| 3 | [PR: Draft → Review → Merge](docs/git-lessons/03-pr-draft-merge.md) | draft PRs, squash merge |
| 4 | [CI](docs/git-lessons/04-ci.md) | GitHub Actions, branch protection |
| 5 | [Release: PyPI, Server, Tag](docs/git-lessons/05-release-pypi-deploy.md) | versioning, tagging, publishing |
| 6 | [Agents and Claude Code](docs/git-lessons/06-agents-claude-code.md) | `claude -p`, agentic PRs |

Start with [the overview](docs/git-lessons/00-overview.md) for setup instructions. Facilitators see [the facilitator guide](docs/git-lessons/00-facilitator-guide.md).

### MCP Reference

- [MCP Guide](docs/MCP_GUIDE.md) — full protocol reference (JSON-RPC, transports, tools/resources/prompts, auth)
- [FastMCP Guide](docs/FASTMCP_GUIDE.md) — Python framework for building MCP servers and clients

### Claude Code Skill

`.claude/skills/property-report/` — a Claude Code skill for UK residential property analysis. Produces comparable sales, EPC, rental yield, stamp duty, and a BUY/WATCH/PASS verdict using live Land Registry and Rightmove data. Requires a connected BOUCH property MCP server.
