# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

A training and resource repository. It contains:
- A 6-lesson Git workflow course (`docs/git-lessons/`)
- MCP protocol and FastMCP framework reference guides (`docs/`)
- A Claude Code skill for UK property analysis (`.claude/skills/property-report/`)

There is no application to build or run. No package config, no test suite.

## Git lessons

The course teaches practical git workflow using `tally` — a tiny Python CLI — as the vehicle. `tally` lives in a **separate repo** that learners fork. The lessons here are facilitator and learner-facing docs only.

Lesson sequence: branch → stash → PRs → CI → releases → agentic PRs (Claude Code).

The facilitator guide (`docs/git-lessons/00-facilitator-guide.md`) contains the exact prompts to give Claude Code at each lesson stage.

## Property report skill

`.claude/skills/property-report/` is a full Claude Code skill with:

- **SKILL.md** — skill definition and workflow (read this first)
- **scripts/** — four Python scripts run sequentially: `normalise_rents.py` → `compute_yield.py` → `underwrite.py` → `render_report.py`
- **assets/** — `report-template.md` and `underwriting-defaults.json` (thresholds editable per client)
- **references/** — EPC nuance, stamp duty 2026 bands, yield benchmarks, rental normalisation rationale

The skill requires a connected BOUCH property MCP server. Data flows through `/tmp/property-report-*.json` files between script steps. The scripts are deterministic post-processors — MCP tools gather raw data, scripts do the maths and rendering.

Three lanes govern which tools to call: Lane A (specific address, full report), Lane B (postcode-only, area scan), Lane C (quick stat). Default to Lane B for vague postcode queries.

## MCP guides

`docs/MCP_GUIDE.md` covers the raw JSON-RPC protocol. `docs/FASTMCP_GUIDE.md` covers the FastMCP Python framework (`@mcp.tool`, `@mcp.resource`, `@mcp.prompt` decorators). These are standalone reference docs — no running servers here.
