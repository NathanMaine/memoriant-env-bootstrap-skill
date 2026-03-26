# Security Policy

## Supported Versions

| Version | Supported |
|---------|-----------|
| 1.0.x   | Yes       |

## Scope

`memoriant-env-bootstrap-skill` is a Claude Code plugin that scans project directories for stack signal files and generates shell scripts and Markdown documentation. It contains no executable server code, no network calls, no authentication logic, and no credential handling.

## What This Plugin Does

- Reads project directory structure and specific signal files (e.g., `requirements.txt`, `package.json`, `.env.example`) from the local filesystem.
- Writes a `bootstrap.sh` shell script and `ONBOARDING.md` Markdown file to a user-specified output directory.
- Does not execute any generated scripts.
- Does not read `.env` files that may contain credentials — only reads `.env.example` or `.env.template` placeholder files.

It does **not**:
- Execute code from the scanned project.
- Make network requests.
- Store or transmit project file contents outside the local filesystem.
- Require API keys or credentials of any kind.
- Write credentials or secrets into generated files.

## Credential Safety

The plugin explicitly avoids reading `.env` files. When a `.env.example` or `.env.template` file is found, the generated `bootstrap.sh` copies it to `.env` and instructs the developer to fill in real values manually. No actual credential values are ever written into generated files.

## Generated Script Safety

Every generated `bootstrap.sh` uses `set -euo pipefail` to exit on error and avoid silent failures. Scripts contain only generic, open-source commands. Company-internal commands, hardcoded secrets, and proprietary tooling are never included.

## Reporting a Vulnerability

If you discover a security concern in this plugin — for example, a path traversal issue in directory scanning, credential leakage in generated scripts, or prompt injection in skill instruction files — please report it responsibly.

**Contact:** Open a [GitHub Security Advisory](https://github.com/NathanMaine/memoriant-env-bootstrap-skill/security/advisories/new) on this repository.

Please include:
- A clear description of the issue.
- Steps to reproduce.
- The potential impact.

Do not open a public issue for security vulnerabilities.

## Response Timeline

- Acknowledgement within 5 business days.
- Assessment and patch (if warranted) within 30 days.

## Audit Notes

All skill and agent definitions in this plugin are plain Markdown files. They contain no executable code, no shell scripts, no binary blobs, and no network-facing logic. They are safe to audit as plain text.
