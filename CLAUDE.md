# Home Assistant Configuration Management

This repository manages Home Assistant configuration files with automated validation, testing, and deployment.

## Before Making Changes

**Always consult the latest Home Assistant documentation** at https://www.home-assistant.io/docs/ before suggesting configurations, automations, or integrations. HA updates frequently and syntax/features change between versions.

## Project Structure

- `config/` - Home Assistant configuration files synced from the HA instance
- `tools/` - Python validators, entity explorer, test runner
- `tests/` - pytest suite for the validators
- `venv/` - Python virtual environment (gitignored)
- `Makefile` - `pull` / `push` / `validate` / `entities` targets
- `Makefile.dev` - dev workflow targets (`dev-test`, `dev-format`, `dev-workflow`)
- `.env` - HA host, SSH, token (gitignored; see `.env.example`)
- `.claude-code/` - hooks (post-edit, pre-push) and project settings
- `.rsync-excludes-pull`, `.rsync-excludes-push` - rsync filters per direction

## Rsync Architecture

This project uses **two separate exclude files** for different sync operations:

| File | Used By | Purpose |
|------|---------|---------|
| `.rsync-excludes-pull` | `make pull` | Less restrictive |
| `.rsync-excludes-push` | `make push` | More restrictive |

**Why separate files?**
- `make pull` downloads most files including `.storage/` (excluding sensitive auth files) for local reference
- `make push` writes never overwrites HA's runtime state (`.storage/`)

**Gotcha — auth file exclude pattern:** Both exclude files must match `.storage/auth` (flat file) AND `.storage/auth/` (dir form). Some HA installs ship `auth` as a single file; the dir-only pattern silently leaks credentials. The same trap applies to other `.storage/` entries where HA's layout varies between versions.

## What This Repo Can and Cannot Manage

### SAFE TO MANAGE (YAML files)
- `automations.yaml` - Automation definitions
- `scenes.yaml` - Scene definitions
- `scripts.yaml` - Script definitions
- `configuration.yaml` - Main configuration
- `secrets.yaml` - Secret values

### NEVER MODIFY LOCALLY (Runtime State)
These files in `.storage/` are managed by Home Assistant at runtime. Local modifications will be **overwritten** by HA on restart or ignored entirely.

### Entity/Device Changes (Manual Only)

Do not change entities or devices programmatically from this repo. If changes are
needed, make them manually in the Home Assistant UI:
- Settings → Devices & Services → Entities → Edit

### Reloading After YAML Changes
- Automations: `POST /api/services/automation/reload`
- Scenes: `POST /api/services/scene/reload`
- Scripts: `POST /api/services/script/reload`

## Workflow Rules

### Before Making Changes
1. Run `make pull` to ensure local files are current
2. Identify if the change affects YAML files or `.storage/` files
3. YAML files → edit locally, then `make push`
4. `.storage/` files → use the HA UI only (manual changes)

### Before Running `make push`
1. Validation runs automatically - do not push if validation fails
2. Only YAML configuration files will be synced (`.storage/` is protected)

### After `make push`
1. Reload the relevant HA components (automations, scenes, scripts)
2. Verify changes took effect in HA

### Pull/Push Discipline
- **Pull only at session start, never between edits.** `make pull` runs `rsync --delete` and silently overwrites uncommitted local edits. If you've started editing, finish and `make push` before pulling again.
- The HA host is the source of truth for `.storage/`, dashboards, and anything HA mutates at runtime.

## Available Commands

### Configuration Management
- `make pull` - Pull latest config from Home Assistant instance
- `make push` - Push local config to Home Assistant (with validation)
- `make backup` - Create backup of current config
- `make validate` - Run all validation tests

### Validation Tools
- `python tools/run_tests.py` - Run complete validation suite
- `python tools/yaml_validator.py` - YAML syntax validation only
- `python tools/reference_validator.py` - Entity/device reference validation
- `python tools/ha_official_validator.py` - Official HA configuration validation

### Entity Discovery Tools
- `make entities` - Explore available Home Assistant entities
- `python tools/entity_explorer.py` - Entity registry parser and explorer
  - `--search TERM` - Search entities by name, ID, or device class
  - `--domain DOMAIN` - Show entities from specific domain (e.g., climate, sensor)
  - `--area AREA` - Show entities from specific area
  - `--full` - Show complete detailed output

## Validation System

A layered suite that runs both Home Assistant's own validation and repo-specific checks. Goal: never save or push YAML that fails validation.

1. **YAML Syntax Validation** - Ensures proper YAML syntax with HA-specific tags
2. **Entity Reference Validation** - Checks that all referenced entities/devices exist
3. **Official HA Validation** - Uses Home Assistant's own validation tools

### Automated Validation Hooks

- **Post-Edit Hook**: Runs validation after editing any YAML files in `config/`
- **Pre-Push Hook**: Validates configuration before pushing to Home Assistant
- **Blocks invalid pushes**: Prevents uploading broken configurations

## Home Assistant Instance Details

- **Host / SSH user / token**: configured in `.env` (see `.env.example`). SSH key resolution typically via `~/.ssh/config`.
- **Remote config path**: `/config/` (standard HA path).

## Entity Registry

The system tracks entities across these domains:
- alarm_control_panel, binary_sensor, button, camera, climate
- device_tracker, event, image, light, lock, media_player
- number, person, scene, select, sensor, siren, switch
- time, tts, update, vacuum, water_heater, weather, zone

## Development Workflow

1. **Pull Latest**: `make pull` to sync from HA
2. **Edit Locally**: Modify files in `config/` directory
3. **Auto-Validation**: Hooks automatically validate on edits
4. **Test Changes**: `make validate` for full test suite
5. **Deploy**: `make push` to upload (blocked if validation fails)

## Important Notes

- **Blueprint files** use `!input` tags which are normal and expected
- **Secrets are skipped** during validation for security
- All python tools require the venv: `source venv/bin/activate && python <tool_path>`

## Troubleshooting

### Validation Fails
1. Check YAML syntax errors first
2. Verify entity references exist in `.storage/` files
3. Run individual validators to isolate issues
4. Check HA logs if official validation fails

### SSH Issues
1. Verify SSH key permissions: `chmod 600 ~/.ssh/your_key`
2. Test connection: `ssh your_homeassistant_host`
3. Check SSH config in `~/.ssh/config`

### Missing Dependencies
1. Activate venv: `source venv/bin/activate`
2. Install requirements: `pip install -r requirements-dev.txt`

## Security

- `config/secrets.yaml` and `config/.storage/` are gitignored — never commit them
- Validators skip `secrets.yaml` (would otherwise leak in error messages)

## Entity Naming Convention

This Home Assistant setup uses a **standardized entity naming convention** for multi-location deployments:

### **Format: `location_room_device_sensor`**

**Structure:**
- **location**: `home`, `office`, `cabin`, etc.
- **room**: `basement`, `kitchen`, `living_room`, `main_bedroom`, `guest_bedroom`, `driveway`, etc.
- **device**: `motion`, `heatpump`, `sonos`, `lock`, `vacuum`, `water_heater`, `alarm`, etc.
- **sensor**: `battery`, `tamper`, `status`, `temperature`, `humidity`, `door`, `running`, etc.

### **Examples:**
```
binary_sensor.home_basement_motion_battery
binary_sensor.home_basement_motion_tamper
media_player.home_kitchen_sonos
media_player.office_main_bedroom_sonos
climate.home_living_room_heatpump
climate.office_living_room_thermostat
lock.home_front_door_august
sensor.office_driveway_camera_battery
vacuum.home_roborock
vacuum.office_roborock
```

### **Benefits:**
- **Clear location identification** - no ambiguity between properties
- **Consistent structure** - easy to predict entity names
- **Automation-friendly** - simple to target location-specific devices
- **Scalable** - supports additional locations or rooms

### **Implementation:**
- All location-based entities follow this convention
- Legacy entities have been systematically renamed
- New entities should follow this pattern
- Vendor prefixes (aquanta_, august_, etc.) are replaced with descriptive device names

### **Claude Code Integration:**
- When creating automations, always ask the user for input if there are multiple choices for sensors or devices
- Use the entity explorer tools to discover available entities before writing automations
- Follow the naming convention when suggesting entity names in automations
