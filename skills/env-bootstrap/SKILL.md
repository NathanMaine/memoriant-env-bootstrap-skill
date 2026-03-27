---
name: env-bootstrap
description: Scan a project folder for language and dependency signals, then generate a bootstrap.sh setup script and an ONBOARDING.md checklist so new contributors can set up and start working on day one.
---

# Environment Bootstrap Skill

## Purpose

This skill teaches Claude Code to act as a developer environment bootstrapping wizard: scan a project directory for clues about its tech stack, infer the correct setup steps, and produce a self-contained `bootstrap.sh` script plus a human-readable `ONBOARDING.md` checklist. Based on `NathanMaine/devex-env-bootstrap-agent`.

The goal is to eliminate the "days of setup" problem for new contributors. The generated files use only generic, open-source commands — no internal secrets, no company-specific tooling.

---

## When to Use This Skill

- A new developer joins the team and asks "how do I get this running?"
- A project lacks setup documentation and contributors are guessing at the steps.
- After a significant stack change (new package manager, new language version, new tools) that makes the old setup instructions wrong.
- As a pre-open-source checklist: "Is this project easy for a stranger to set up?"
- In CI: generate bootstrap validation as part of repository health checks.

---

## Prerequisites

1. Access to the project directory (local filesystem).
2. The project should contain at least one of the detector signals listed below.
3. No installation required — the skill performs file scanning only.

---

## Stack Detection Signals

### Python

| Signal File | Inferred Setup |
|-------------|----------------|
| `requirements.txt` | `python -m venv .venv && source .venv/bin/activate && pip install -r requirements.txt` |
| `pyproject.toml` with `[tool.poetry]` | `poetry install` |
| `pyproject.toml` with `[build-system]` | `pip install -e .` |
| `Pipfile` | `pipenv install` |
| `setup.py` | `pip install -e .` |
| `.python-version` | Read version, instruct to use `pyenv install <version>` |

### Node.js / JavaScript / TypeScript

| Signal File | Inferred Setup |
|-------------|----------------|
| `package.json` + `package-lock.json` | `npm install` |
| `package.json` + `yarn.lock` | `yarn install` |
| `package.json` + `pnpm-lock.yaml` | `pnpm install` |
| `.nvmrc` | `nvm use` |
| `.node-version` | Read version, instruct `nvm install <version>` |

### Rust

| Signal File | Inferred Setup |
|-------------|----------------|
| `Cargo.toml` | `cargo build` |
| `Cargo.lock` | `cargo build --locked` |

### Go

| Signal File | Inferred Setup |
|-------------|----------------|
| `go.mod` | `go mod download && go build ./...` |

### Java / Kotlin

| Signal File | Inferred Setup |
|-------------|----------------|
| `pom.xml` | `mvn install` |
| `build.gradle` or `build.gradle.kts` | `./gradlew build` |

### Ruby

| Signal File | Inferred Setup |
|-------------|----------------|
| `Gemfile` | `bundle install` |
| `.ruby-version` | Read version, instruct `rbenv install <version>` |

### Docker / Containers

| Signal File | Inferred Setup |
|-------------|----------------|
| `Dockerfile` | `docker build -t <project-name> .` |
| `docker-compose.yml` or `compose.yaml` | `docker compose up -d` |

### Environment Variables

| Signal File | Action |
|-------------|--------|
| `.env.example` | Copy to `.env`: `cp .env.example .env` — instruct user to fill in values |
| `.env.template` | Same as above |

### Database Migrations

| Signal File / Pattern | Action |
|-----------------------|--------|
| `alembic.ini` | `alembic upgrade head` |
| `migrations/` directory + `manage.py` | `python manage.py migrate` |
| `db/migrate/` (Rails) | `rails db:migrate` |
| `flyway.conf` | `flyway migrate` |

---

## Step-by-Step Methodology

### Step 1 — Scan the project directory

Walk the top-level directory and one level deep. Collect all signal files. Do not recurse deeper than two levels for detection — focus on the root and immediate subdirectories.

### Step 2 — Detect the stack

For each signal file found, record:
- Language/runtime
- Dependency manager
- Required tool versions (read from `.python-version`, `.nvmrc`, etc.)
- Environment variable requirements (from `.env.example`)
- Container orchestration (Dockerfile, compose)
- Database migration needs

If multiple languages are detected, record all of them — the project is polyglot.

### Step 3 — Infer setup order

Determine the correct sequencing for `bootstrap.sh`:

1. Tool version setup (pyenv, nvm, rbenv) — before anything else.
2. Dependency installation (pip, npm, cargo, go mod, bundle).
3. Environment file setup (`.env` from `.env.example`).
4. Container build/start (if Dockerfile/compose detected).
5. Database migrations (if migration signals detected).
6. Test run (if test runner detected: `pytest`, `npm test`, `cargo test`, `go test ./...`).

### Step 4 — Generate bootstrap.sh

Write a `bootstrap.sh` script with:
- A shebang: `#!/usr/bin/env bash`
- `set -euo pipefail` for safety.
- Descriptive `echo` statements before each section.
- Only generic, open-source commands. No internal scripts, no company-specific tooling, no secrets.
- A final success message.

Example structure:

```bash
#!/usr/bin/env bash
set -euo pipefail

echo "==> Checking Python version..."
python3 --version

echo "==> Creating virtual environment..."
python3 -m venv .venv

echo "==> Activating virtual environment..."
source .venv/bin/activate

echo "==> Installing dependencies..."
pip install -r requirements.txt

echo "==> Setting up environment variables..."
if [ ! -f .env ]; then
  cp .env.example .env
  echo "    .env created from .env.example — fill in required values before running the app."
fi

echo "==> Running tests to verify setup..."
pytest

echo ""
echo "Bootstrap complete. Run 'source .venv/bin/activate' to activate the environment."
```

### Step 5 — Generate ONBOARDING.md

Write a `ONBOARDING.md` checklist with:
- A brief intro paragraph (what the project is, one sentence if inferable from README or package.json).
- Prerequisites section (tools to install manually: Python, Node, Docker, etc.).
- Quick start section (run `./bootstrap.sh`).
- Manual steps section (things that cannot be automated: filling in `.env` values, requesting API keys, getting database credentials).
- Verify section (how to confirm the setup worked: run tests, open the app, etc.).
- Troubleshooting section (common issues for the detected stack).

### Step 6 — Report to user

After writing both files, report:

```
Environment Bootstrap Complete
──────────────────────────────
Stack detected     : Python 3.11, Node.js 20 (npm), Docker
Files generated    :
  bootstrap.sh     — run this to set up the environment
  ONBOARDING.md    — human checklist for new contributors

Manual steps required:
  1. Fill in values in .env (copied from .env.example)
  2. Request API key for [SERVICE] from the team
```

---

## Input Format

### Natural language

```
Generate a bootstrap script for this project
```

```
Write an onboarding guide for a new developer joining this repo
```

```
Set up bootstrap.sh and ONBOARDING.md for ./my_project
```

### Skill invocation

```
/env-bootstrap --project-path ./my_project --output-dir ./my_project
```

### Parameters

| Parameter | Default | Description |
|-----------|---------|-------------|
| `project_path` | `.` | Root of the project to scan |
| `output_dir` | same as `project_path` | Where to write `bootstrap.sh` and `ONBOARDING.md` |
| `overwrite` | `false` | Overwrite existing bootstrap.sh / ONBOARDING.md |

---

## Output Format

1. `bootstrap.sh` — executable shell script in `output_dir`.
2. `ONBOARDING.md` — human-readable Markdown checklist in `output_dir`.
3. Console summary as shown above.

---

## Error Handling

| Situation | Response |
|-----------|----------|
| No signal files found | Report that no known stack was detected. Ask the user to describe the tech stack or point to a signal file. |
| `project_path` does not exist | Report the path. Ask the user to confirm. |
| `bootstrap.sh` already exists and `overwrite=false` | Report it exists. Ask before overwriting. |
| Multiple conflicting signals (e.g., both `requirements.txt` and `Pipfile`) | Include both in the script with a comment explaining the ambiguity. Ask the user which is canonical. |
| `.env.example` does not exist but `.env` does | Warn that `.env` may contain secrets. Do not read `.env`. Suggest creating `.env.example`. |

---

## Examples

### Example 1 — Python project

User: "Write a bootstrap script for this project"

Detected: `requirements.txt`, `.python-version` (3.11), `.env.example`, `pytest` in requirements.

Output: `bootstrap.sh` that creates a venv, installs requirements, copies `.env.example`, and runs pytest. `ONBOARDING.md` with pyenv install instructions.

### Example 2 — Node + Docker polyglot

User: "Generate ONBOARDING.md and bootstrap.sh for ./frontend"

Detected: `package.json` + `pnpm-lock.yaml`, `.nvmrc` (20), `docker-compose.yml`, `.env.example`.

Output: `bootstrap.sh` that runs `nvm use`, `pnpm install`, copies `.env`, and starts Docker Compose. `ONBOARDING.md` includes manual Docker credential step.

### Example 3 — Rust project

User: "New hire starts Monday — generate their setup docs"

Detected: `Cargo.toml`, `Cargo.lock`.

Output: `bootstrap.sh` with `cargo build --locked`. `ONBOARDING.md` with `rustup` install instructions and `cargo test` verification step.

---

## Version History

| Version | Date | Notes |
|---------|------|-------|
| 1.0.0 | 2026-03-25 | Initial release — multi-stack detection, bootstrap.sh + ONBOARDING.md generation |
