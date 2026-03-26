# memoriant-env-bootstrap-skill

![Version](https://img.shields.io/badge/version-1.0.0-blue)
![License](https://img.shields.io/badge/license-MIT-green)
![Platform](https://img.shields.io/badge/platform-Claude%20Code%20%7C%20Codex%20CLI%20%7C%20Gemini%20CLI-lightgrey)

A Claude Code plugin that eliminates the "days of setup" problem. Scans a project directory for stack signals, then generates a portable `bootstrap.sh` setup script and `ONBOARDING.md` checklist so new contributors can set up and start working in minutes.

Built on the methodology of [`NathanMaine/devex-env-bootstrap-agent`](https://github.com/NathanMaine/devex-env-bootstrap-agent).

---

## What It Does

1. Walks the project directory collecting signal files: `requirements.txt`, `package.json`, `Cargo.toml`, `go.mod`, `Dockerfile`, `.env.example`, and more.
2. Infers the full tech stack — supports Python, Node.js/TypeScript, Rust, Go, Ruby, Java/Kotlin, Docker, and common database migration tools.
3. Determines the correct setup order (version managers first, dependencies second, env file, containers, migrations, tests).
4. Generates `bootstrap.sh` — a safe, generic shell script with `set -euo pipefail`, descriptive echo statements, and only open-source commands.
5. Generates `ONBOARDING.md` — prerequisites, quick start, manual steps that cannot be automated, verification, and troubleshooting.
6. Never reads `.env` files. Never writes secrets. Never overwrites existing files without asking.

---

## Installation

### Claude Code

```bash
claude mcp add memoriant-env-bootstrap-skill
```

Or clone locally:

```bash
git clone https://github.com/NathanMaine/memoriant-env-bootstrap-skill ~/.claude/plugins/memoriant-env-bootstrap-skill
```

### Codex CLI

```bash
codex install NathanMaine/memoriant-env-bootstrap-skill
```

### Gemini CLI

```bash
gemini extension install NathanMaine/memoriant-env-bootstrap-skill
```

---

## Usage

### In Claude Code (natural language)

```
Generate a bootstrap script for this project
```

```
Write an onboarding guide for a new developer joining this repo
```

```
Set up bootstrap.sh and ONBOARDING.md for ./my_project
```

### Via skill invocation

```
/env-bootstrap --project-path ./my_project --output-dir ./my_project
```

### Via Codex CLI

```bash
codex run env-bootstrap-agent --var project_path=./my_project
```

---

## Plugin Structure

```
memoriant-env-bootstrap-skill/
├── .claude-plugin/
│   └── plugin.json              # Plugin manifest
├── skills/
│   └── env-bootstrap/
│       └── SKILL.md             # Full methodology for Claude Code
├── agents/
│   └── env-bootstrap-agent.md  # Autonomous agent definition
├── AGENTS.md                    # Codex CLI agent definitions
├── gemini-extension.json        # Gemini CLI extension manifest
├── SECURITY.md                  # Security policy
├── README.md                    # This file
└── LICENSE                      # MIT
```

---

## Stack Detection Coverage

| Language / Tool | Signal Files Detected |
|-----------------|----------------------|
| Python | `requirements.txt`, `pyproject.toml`, `Pipfile`, `setup.py`, `.python-version` |
| Node.js / TypeScript | `package.json`, `package-lock.json`, `yarn.lock`, `pnpm-lock.yaml`, `.nvmrc` |
| Rust | `Cargo.toml`, `Cargo.lock` |
| Go | `go.mod` |
| Ruby | `Gemfile`, `.ruby-version` |
| Java / Kotlin | `pom.xml`, `build.gradle`, `build.gradle.kts` |
| Docker | `Dockerfile`, `docker-compose.yml`, `compose.yaml` |
| Env vars | `.env.example`, `.env.template` |
| DB migrations | `alembic.ini`, `manage.py migrations/`, `db/migrate/`, `flyway.conf` |

---

## Sample Output

### bootstrap.sh

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
  echo "    .env created — fill in required values before running the app."
fi

echo "==> Running tests to verify setup..."
pytest

echo ""
echo "Bootstrap complete. Run 'source .venv/bin/activate' to activate the environment."
```

### Console Summary

```
Environment Bootstrap Complete
──────────────────────────────
Stack detected     : Python 3.11, Docker
Files generated    :
  bootstrap.sh     — run this to set up the environment
  ONBOARDING.md    — human checklist for new contributors

Manual steps required:
  1. Fill in values in .env (copied from .env.example)
  2. Request DATABASE_URL from the team
```

---

## Source Reference

Derived from [`NathanMaine/devex-env-bootstrap-agent`](https://github.com/NathanMaine/devex-env-bootstrap-agent), a Python CLI prototype that scans projects and generates `bootstrap.sh` and `ONBOARDING.md` files using stack detection heuristics.

---

## License

MIT — see [LICENSE](LICENSE).
