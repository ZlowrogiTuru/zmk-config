# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

This is a ZMK firmware configuration for a **Redox** split keyboard, running on **nrfmicro/nrf52840** controllers. ZMK is a Zephyr-based keyboard firmware. There is no local build toolchain — firmware is compiled exclusively via GitHub Actions.

## Building

Firmware builds are triggered automatically on push/PR via `.github/workflows/build.yml`, which delegates to the upstream ZMK reusable workflow. The `build.yaml` matrix defines three targets:

- `nrfmicro/nrf52840/zmk` + `redox_left`
- `nrfmicro/nrf52840/zmk` + `redox_right`
- `nrfmicro/nrf52840/zmk` + `settings_reset`

To trigger a build, push to `main` or open a PR. Download firmware artifacts from the GitHub Actions run.

## Repository Structure

```
config/
  redox.keymap   # Main keymap (DeviceTree/ZMK syntax) — the primary file to edit
  redox.conf     # Kconfig options (e.g. ZMK_MOUSE=y)
  west.yml       # ZMK dependency manifest (pins zmk@main)
  redox.json     # Physical layout definition for Keymap Editor GUI
build.yaml       # GitHub Actions build matrix
layout.vil       # Vial/QMK legacy layout export (reference only, not used by ZMK)
```

## Keymap Architecture

The keymap (`config/redox.keymap`) uses ZMK's DeviceTree `.keymap` format with 7 layers:

| ZMK Index | Name          | Purpose                                      |
|-----------|---------------|----------------------------------------------|
| 0         | `default_layer` | Main QWERTY layer                          |
| 1         | `layers`      | Layer switcher (momentary, accessed via `&mo 1`) |
| 2         | `symbols`     | Symbols (left) + numpad (right)              |
| 3         | `game`        | Gaming layer (WASD-focused)                  |
| 4         | `media`       | Mouse movement, scroll, media controls, workspace switching |
| 5         | `settings`    | Function keys F1–F12, bootloader, sys_reset  |
| 6         | `macros`      | Arrow macros, Bluetooth controls, ZMK Studio unlock |

### Key Behaviors Defined

- **`qht`** (`q_hold_tap`): tap=Q, hold=`LC(GRAVE)` — terminal dropdown shortcut
- **`td_b`** (`tap_dance_b`): 1 tap=B (with `&lt 1` hold), 2 taps=`TO(4)` (media layer)
- **`td_ctrl`** (`tap_dance_ctrl`): 1 tap/hold=LCTRL, 2 taps=ENTER
- **Macros**: `Arrow` (=>), `ArrowFn` (()=>{}), `ArrowRet` (=>(){}), `fllFN`, `OmarchyManu` (LAlt+Space)

### Combos (key-position based)

- `A+S` → ESC
- `Q+W` → 1, `W+E` → 2, `E+R` → 3, `R+T` → 4, `Q+T` → 5

### Kconfig

`redox.conf` enables `CONFIG_ZMK_MOUSE=y` (required for mouse keys in the media layer). RGB underglow lines are present but commented out.

## Editing Guidelines

- The primary file to edit is `config/redox.keymap`.
- `redox.json` describes the physical key positions for use with the [Keymap Editor](https://nickcoutsos.github.io/keymap-editor/) GUI — update this only if the physical layout changes.
- `layout.vil` is a legacy Vial export from a previous QMK firmware; it serves as a historical reference for the original layout intent and is not used by ZMK.
- `west.yml` pins ZMK to `main` branch; change `revision` to pin to a specific commit if needed.
- To add a new behavior or macro, define it in the `/ { behaviors { ... }; };` or `/ { macros { ... }; };` blocks before using it in `keymap`.
- ZMK key codes use `&kp`, layer momentary via `&mo N`, layer toggle via `&to N`, layer-tap via `&lt N KEY`, mod-tap via `&mt MOD KEY`.
