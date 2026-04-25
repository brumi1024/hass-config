# hass-config

Personal Home Assistant configuration with Claude Code-assisted authoring,
validation, and deployment.

## Prerequisites

- macOS
- Python 3.12+
- SSH access to the HA host (configured in `.env` — see `.env.example`)
- `rsync`, `make`

## Setup

```bash
./setup-mac.sh
```

Creates the venv, installs dependencies, and primes pre-commit hooks.

## Daily commands

| Command | What it does |
| --- | --- |
| `make pull` | Sync HA → local. Run at session start; do NOT run between edits. |
| `make push` | Validate then sync local → HA. Blocked if validation fails. |
| `make validate` | Run the full validation suite locally. |
| `make entities ARGS='--search <term>'` | Browse the entity registry. |

## Notes

- See `CLAUDE.md` for the agent-facing instructions and naming conventions.
- See `AGENTS.md` for the broader contributor guide.
- Source of truth for live config is the HA host. Local files reflect the last `make pull`.
