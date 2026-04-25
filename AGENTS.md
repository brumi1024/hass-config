# Repository Guidelines

## Project Structure & Module Organization

This repository manages a Home Assistant configuration. Primary configuration lives in `config/`, with dashboards under `config/dashboards/home/`, reusable dashboard templates under `config/dashboards/home/templates/`, packages under `config/packages/`, and custom web assets under `config/www/`. Python validation and utility scripts live in `tools/`. Tests live in `tests/`, with a few legacy top-level test files retained for reference. Planning notes and design specs are in `docs/superpowers/`.

## Build, Test, and Development Commands

- `make setup` creates the local Python virtual environment and installs runtime validation dependencies.
- `make validate` or `make test` runs the Home Assistant validation suite through `tools/run_tests.py`.
- `make pull` syncs configuration from the Home Assistant host, then validates it.
- `make push` validates local config, syncs YAML-safe changes to Home Assistant, and reloads configuration.
- `make entities ARGS='--search temp'` explores known Home Assistant entities.
- `make -f Makefile.dev dev-format` formats Python with Black and isort.
- `make -f Makefile.dev dev-test` runs pytest tests.
- `make -f Makefile.dev dev-workflow` formats, lints, type-checks, and tests Python tooling.

## Coding Style & Naming Conventions

Python targets 3.12+, uses 4-space indentation, Black line length 88, and isort’s Black profile. Keep Python modules in `tools/` snake_case and tests named `test_*.py`. YAML uses 2-space indentation and Home Assistant conventions; do not reformat generated or runtime-managed files unnecessarily. Dashboard view files use descriptive kebab-case names such as `room-living-room.yaml`.

## Testing Guidelines

Pytest discovers tests in `tests/` using `test_*.py` and `test_*` functions. The default pytest config measures coverage for `tools/` and targets 80% coverage; CI also runs focused rsync exclude and reference validator tests. For YAML changes, run `make validate`; for Python tooling changes, run `make -f Makefile.dev dev-test` or the full `dev-workflow`.

## Commit & Pull Request Guidelines

Recent commits use concise, scoped prefixes such as `P6 fix: ...`, `P6 docs: ...`, or `P6-B: ...`. Follow that pattern when working inside an active phase, and keep the subject imperative and specific. Pull requests should describe the changed Home Assistant behavior, list validation commands run, link related issues or plans, and include screenshots for dashboard UI changes.

## Security & Configuration Tips

Configure secrets and host settings through `.env` and Home Assistant secret files; do not commit credentials. Treat `.storage/` as runtime state managed by Home Assistant, not source-controlled configuration. Before changing integration syntax, check current Home Assistant documentation because supported options change frequently.
