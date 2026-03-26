# AGENTS.md — Codex CLI Agent Definitions

This file follows the [Codex CLI AGENTS.md convention](https://github.com/openai/codex).

## memoriant-env-bootstrap-skill

**Purpose:** Scan a project for stack signals, generate a portable `bootstrap.sh` setup script and `ONBOARDING.md` checklist so new contributors can start working on day one.

**Activation:** `codex run env-bootstrap-agent`

### Agent Behavior

The agent walks a project directory collecting signal files (requirements.txt, package.json, Cargo.toml, go.mod, Dockerfile, .env.example, etc.), infers the full tech stack and correct setup order, and writes two files: a shell script that automates every automatable step, and a Markdown checklist covering prerequisites, verification, and manual steps that cannot be scripted. It never reads `.env` files, never writes to source code, and never overwrites existing files without confirmation.

### Inputs

| Name | Type | Required | Default | Description |
|------|------|----------|---------|-------------|
| `project_path` | string (path) | No | `.` | Root of the project to scan |
| `output_dir` | string (path) | No | same as `project_path` | Where to write `bootstrap.sh` and `ONBOARDING.md` |
| `overwrite` | boolean | No | `false` | Overwrite existing bootstrap files |

### Outputs

| Name | Description |
|------|-------------|
| `bootstrap.sh` | Executable shell script that sets up the development environment |
| `ONBOARDING.md` | Human-readable Markdown checklist for new contributors |
| Console summary | Stack detected, files written, manual steps required |

### Agent File

See [`agents/env-bootstrap-agent.md`](agents/env-bootstrap-agent.md) for the full system prompt.

### Skill File

See [`skills/env-bootstrap/SKILL.md`](skills/env-bootstrap/SKILL.md) for the full methodology.

---

## Usage with Codex CLI

```bash
# Basic run — scan current directory
codex run env-bootstrap-agent

# Custom project path
codex run env-bootstrap-agent --var project_path=./my_project

# Write output to a different directory
codex run env-bootstrap-agent --var project_path=./my_project --var output_dir=./docs

# Overwrite existing files
codex run env-bootstrap-agent --var project_path=./my_project --var overwrite=true
```

---

## Stack Detection Coverage

| Language / Tool | Signal Files |
|-----------------|-------------|
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

## Integration Notes

- Compatible with Claude Code, Codex CLI, and any agent runtime that supports Markdown-based agent definitions.
- No API keys or external services required — the agent performs file scanning only.
- Safe for CI pipelines: generated scripts only use open-source commands.
- The generated `bootstrap.sh` uses `set -euo pipefail` — it exits on any error, making it safe to run in automated environments.
