# Repository Guidelines

## Project Structure & Source Of Truth

This repo manages Home Assistant configuration synced from an HA instance. Primary configuration lives in `config/`, with dashboards in `config/dashboards/home/`, reusable dashboard templates in `config/dashboards/home/templates/`, packages in `config/packages/`, and web assets in `config/www/`. Python validators, deployment helpers, and entity tools live in `tools/`; pytest coverage lives in `tests/`.

Always check current Home Assistant documentation before suggesting configuration, automation, or integration syntax. HA changes often enough that remembered options can be stale.

## Commands

- `make setup` creates `venv/` and installs runtime validation dependencies.
- `make validate` or `make test` runs `tools/run_tests.py`.
- `make pull` syncs from HA using `.rsync-excludes-pull`, then validates.
- `make push` validates, syncs using `.rsync-excludes-push`, then runs `tools/reload_config.py`.
- `make backup`, `make status`, `make reload`, and `make format-yaml FILES='...'` support routine config work.
- `make entities ARGS='--search temp'` explores known HA entities.
- `make -f Makefile.dev dev-format`, `dev-test`, and `dev-workflow` handle Python formatting, pytest, linting, typing, and tests.

## Workflow Rules

Run `make pull` only at the start of a session. It uses `rsync --delete` and can silently overwrite uncommitted local edits. YAML changes should be edited locally, validated with `make validate`, then deployed with `make push`. Do not change entities or devices programmatically from this repo; use the Home Assistant UI.

## Style & Naming

Python targets 3.12+, uses 4-space indentation, Black line length 88, and isort's Black profile. Keep modules in `tools/` snake_case and tests named `test_*.py` or `*_test.py`. YAML uses 2-space indentation and HA conventions; avoid reformatting generated or runtime-managed files.

For location-based entities, follow `location_room_device_sensor`, for example `binary_sensor.home_basement_motion_battery` or `climate.home_living_room_heatpump`. Use `make entities` or `tools/entity_explorer.py` before writing automations, and ask the user when multiple sensors or devices could satisfy the request.

## Testing

Pytest discovers tests in `tests/` and measures coverage for `tools/` with an 80% threshold. For YAML changes, run `make validate`; for Python tooling changes, run `make -f Makefile.dev dev-test` or the full `dev-workflow`. Use individual validators such as `python tools/yaml_validator.py` or `python tools/reference_validator.py` to isolate failures.

## Security & Runtime State

Configure host, SSH, and token values through `.env` and HA secret files; do not commit credentials. Treat `config/.storage/` as runtime state owned by Home Assistant. Local edits there will be overwritten or ignored, and `make push` must not overwrite HA runtime state. Keep both rsync exclude files matching `.storage/auth` and `.storage/auth/`.
