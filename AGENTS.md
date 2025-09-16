# Repository Guidelines

## Project Structure & Module Organization
- `keyboards/nuphy/halo75_v2/` — Halo75 V2 keyboard definition; `ansi/` holds firmware sources, keymaps, and configs.
- `keyboards/nuphy/halo75_v2/ansi/keymaps/` — individual keymaps (`default/`, `via/`) plus VIA JSON assets.
- `docs/` — engineering write-ups (e.g., `mac_globe_dual_role.md`, English version) for reference.
- Build outputs (e.g., `nuphy_halo75_v2_ansi_via.bin`) land in repo root after compilation.

## Build, Test, and Development Commands
- `qmk compile -kb nuphy/halo75_v2/ansi -km via` — build the VIA-enabled firmware; artifacts appear in `.build/` and root.
- `qmk flash -kb nuphy/halo75_v2/ansi -km via` — compile and flash in one step (board must be in bootloader/DFU mode).
- Use `make git-submodule` if submodules fall out of sync before building.

## Coding Style & Naming Conventions
- Follow QMK conventions: tabs for alignment inside keymaps, spaces for struct/logic formatting, and upper-case custom keycodes (`MAC_GLOBE`).
- Place new keycodes in `enum custom_keycodes` with clear prefixes (e.g., `MAC_`, `SIDE_`).
- Comment sparingly but clarify non-obvious logic (see state machine for `MAC_GLOBE_CTRL`).

## Testing Guidelines
- QMK firmware relies on hardware verification. After building, flash to device and test tap/hold behavior, layer switching, and VIA detection.
- For json assets, load into VIA (Design tab → File → Import) to confirm layout renders and custom keycodes appear.

## Commit & Pull Request Guidelines
- Use descriptive commits that mention the area touched, e.g., `Add dual-role Globe keycode`.
- For pull requests: include summary, testing steps (build/flash outcome), and attach VIA screenshots if UI elements change.
- Reference related GitHub issues (e.g., `Fixes #16651`) where applicable.
