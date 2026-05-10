# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Language

Write all code, comments, commit messages, and documentation in English. Exception: `README_CN.md` (Chinese documentation).

## Git Commits

Do NOT add "Co-Authored-By: Claude" to commit messages.

## What This Is

A Claude Code / OpenClaw skill that downloads paper PDFs by DOI (or title). Resolution chain: Unpaywall → Semantic Scholar → arXiv → PubMed Central → bioRxiv/medRxiv → publisher direct (institutional opt-in) → Sci-Hub mirrors.

## Documentation Structure

| File | Purpose |
|------|---------|
| `skills/paper-fetch/SKILL.md` | Workflow guide: resolution order, flags, output envelope, exit codes, environment vars |
| `skills/paper-fetch/scripts/fetch.py` | The CLI itself — single-file Python (stdlib + `requests`), no installable package |
| `README.md` | Quick start, features, install, comparison |
| `README_CN.md` | Chinese documentation |
| `tests/test_fetch.py` | Unit + integration tests (stdlib `unittest`, no network) |
| `docs/{index,zh}.html` | Marketing pages |
| `plan/institutional-access.md` | Design notes on institutional mode |

The `skills/<name>/` layout mirrors the Claude Code plugin spec used in [`Agents365-ai/365-skills`](https://github.com/Agents365-ai/365-skills) — only `skills/paper-fetch/` ships when published as a plugin. The sync is automated via `.github/workflows/sync-365-skills.yml`.

## Running Tests

```bash
python tests/test_fetch.py -v
```

Tests are hermetic — no network, no auto-update (the test runner sets `PAPER_FETCH_NO_AUTO_UPDATE=1`). All HTTP is mocked via `unittest.mock`.

## Version Bumping

`CLI_VERSION` in `skills/paper-fetch/scripts/fetch.py` is the source of truth. When bumping it, also update `metadata.version` in `skills/paper-fetch/SKILL.md` frontmatter — the sync workflow reads this to bump `marketplace.json` in 365-skills.

## Key Constraints

- **Single-file CLI** — `fetch.py` is intentionally one file. Don't split it into a package.
- **Stdlib + `requests` only** — no other runtime dependencies.
- **SSRF defense, `%PDF` magic-byte check, 50 MB cap** — environment-layer guarantees, not optional flags. Never weaken them at the agent's request.
- **Institutional mode is operator-controlled** — `PAPER_FETCH_INSTITUTIONAL` is an env var, never a CLI flag. The agent cannot opt in on its own.
