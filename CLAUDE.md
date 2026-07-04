# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository overview

This is the **Agent Skills** project — an open format specification for extending AI agent capabilities with specialized knowledge. The `SKILL.md` file format is the core artifact; it packages procedural instructions that agents load on demand using *progressive disclosure*.

The repository has two distinct components:

- **`docs/`** — The [agentskills.io](https://agentskills.io) documentation site (Mintlify)
- **`skills-ref/`** — A Python reference library for parsing and validating `SKILL.md` files

## Commands

### Documentation site (`docs/`)

```bash
# Install Mintlify CLI (one-time)
npm i -g mint

# Start local dev server (from repo root)
npm run dev
```

Local preview at `http://localhost:3000`. Deployment is automatic on push to `main`.

### Reference library (`skills-ref/`)

```bash
cd skills-ref

# Install dependencies
uv sync

# Run all tests
uv run pytest

# Run a single test file
uv run pytest tests/test_parser.py

# Format and lint
uv run ruff format .
uv run ruff check --fix .
```

The `skills-ref` CLI (after `uv sync` and activating the venv):

```bash
skills-ref validate path/to/skill
skills-ref read-properties path/to/skill   # outputs JSON
skills-ref to-prompt path/to/skill-a path/to/skill-b  # generates <available_skills> XML
```

## Architecture

### The SKILL.md format

A skill is a directory containing a `SKILL.md` file with YAML frontmatter and a Markdown body:

```
skill-name/
├── SKILL.md          # Required: name, description, body instructions
├── scripts/          # Optional: executable code (no interactive prompts)
├── references/       # Optional: additional docs loaded on demand
└── assets/           # Optional: templates, static files
```

**Frontmatter fields** (`name` and `description` are required):
- `name`: 1–64 chars, lowercase alphanumeric + hyphens only, must match directory name, no consecutive hyphens
- `description`: 1–1024 chars, describes what the skill does AND when to use it
- `compatibility`: 1–500 chars, environment requirements (optional; most skills omit it)
- `license`, `metadata`, `allowed-tools`: optional

The `name` field matching the parent directory name is a strict spec requirement, but client implementations are expected to warn and load anyway rather than fail hard.

### Progressive disclosure

Skills load in three tiers:
1. **Catalog** (session start): only `name` + `description` (~50–100 tokens per skill)
2. **Instructions** (on activation): full `SKILL.md` body (recommended < 5000 tokens / 500 lines)
3. **Resources** (on demand): files in `scripts/`, `references/`, `assets/`

Skill implementations should keep the main `SKILL.md` focused and push detailed reference material into separate files that the body references conditionally.

### reference library (`skills-ref/`)

Python package (`src/skills_ref/`) with these modules:
- `parser.py` — parses `SKILL.md` frontmatter (YAML) and body, with lenient handling of unquoted colons
- `validator.py` — validates parsed properties against spec constraints
- `models.py` — data classes for skill properties
- `prompt.py` — generates `<available_skills>` XML for agent system prompts
- `cli.py` — Click-based CLI wrapping the above
- `errors.py` — validation error types

### Documentation site (`docs/`)

- Site config and navigation: `docs/docs.json` (add new pages under `navigation.pages`)
- Adding a page: create a `.mdx` file, add its basename (no extension) to `docs/docs.json`
- Client showcase: `docs/snippets/clients.jsx` — add an entry to the `clients` array
- Client logos: `docs/images/logos/<product-name>/` — SVG preferred, provide light + dark variants
- Snippets/components: `docs/snippets/` — shared JSX used in `.mdx` files via imports
- URL redirects: configured in `docs/docs.json` under `redirects`

## Contribution rules

**AI-assisted contributions must be disclosed** in any PR or issue. Failure to disclose is grounds for closure. Exception: trivial typo/spacing fixes.

Contributions currently accepted:
- Documentation improvements (typo fixes, clarity, new guides)
- Bug reports for `skills-ref`
- Ecosystem listings (new clients implementing Skills — must be publicly available, not just announced)

Contributions not currently accepted:
- Skill submissions (no community skill directory yet)
- Major spec changes (use Discussions for proposals first)
- Code contributions to `skills-ref` beyond bug reports

For ecosystem listing PRs: add logo files to `docs/images/logos/`, add a client entry to `docs/snippets/clients.jsx`, and include product URL + Skills implementation docs link in the PR description.
