# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is a ZMK (Zephyr-based Mechanical Keyboard) firmware configuration repository for custom split wireless keyboards. The repository contains configurations for two main keyboard models:
- **Corne Choc Pro BT**: A wireless Corne-style keyboard with Choc switches
- **Piantor Pro BT**: A wireless Piantor keyboard

## Architecture

### Directory Structure
- `/config/`: Main keyboard keymaps and West manifest
  - `west.yml`: West manifest pointing to ZMK upstream
  - `*.keymap`: User-customizable keymaps for each board
- `/boards/arm/`: Board definitions and hardware configurations
  - Each board has its own directory with DTS files, layout definitions, and metadata
- `/.github/workflows/`: GitHub Actions for automatic firmware builds
- `build.yaml`: Build matrix configuration for GitHub Actions

### Build System
The project uses:
- **West**: Zephyr's meta-tool for managing repositories and building
- **GitHub Actions**: Automated firmware building via `zmkfirmware/zmk/.github/workflows/build-user-config.yml@main`
- **CMake**: Build configuration with ZMK Studio support (`-DCONFIG_ZMK_STUDIO=y`)

## Common Development Commands

### Local Development (requires West and Zephyr toolchain)
```bash
# Initialize West workspace (first time only)
west init -l config
west update

# Build firmware for specific board/shield combination
west build -p -d build/left -b piantor_pro_bt_left -- -DSHIELD=nice_view
west build -p -d build/right -b piantor_pro_bt_right -- -DSHIELD=nice_view

# Build with ZMK Studio support
west build -p -d build/left -b piantor_pro_bt_left -- -DSHIELD=nice_view -DCONFIG_ZMK_STUDIO=y

# Flash firmware (when connected via USB)
west flash -d build/left
```

### GitHub Actions Build
```bash
# Push changes to trigger automatic builds
git add .
git commit -m "Update keymap"
git push

# Firmware artifacts will be available in GitHub Actions artifacts
```

## Keymap Configuration

### Keymap Structure
Keymaps are defined in `.keymap` files using Device Tree syntax:
- Layers are defined in the `keymap` node with `compatible = "zmk,keymap"`
- Each layer has a `bindings` property containing the key mappings
- Combos can be defined for multi-key shortcuts
- Behaviors like mod-tap can be configured

### Key Locations
The keyboards use a 42-key layout (3x6+3 per half):
- Main keys: 36 keys (3 rows × 6 columns × 2 halves)
- Thumb keys: 6 keys (3 per side)

### Available Layers (in piantor_pro_bt.keymap)
1. **QWERTY** (default): Standard typing layer
2. **NUMBER**: Number row and navigation
3. **SYMBOL**: Symbols and function keys
4. **bt**: Bluetooth controls and system functions
5. **code**: Programming-specific symbols
6. Additional layers for customization

## Key Features and Configurations

### Bluetooth & Connectivity
- Multiple Bluetooth profiles (BT_SEL 0-4)
- USB and BLE output support
- Settings reset shield for troubleshooting

### Display Support
- Nice!View display shield support
- Display names for each layer

### ZMK Studio
- Enabled via cmake flag: `-DCONFIG_ZMK_STUDIO=y`
- Studio RPC over USB UART snippet
- `studio_unlock` binding for accessing Studio mode

### Mod-Tap Configuration
- Configured with 200ms tapping term
- "tap-preferred" flavor for better typing experience
- Home row mods configured on various keys

## Modifying Keymaps

1. Edit the appropriate `.keymap` file in `/config/`
2. Modify the `bindings` array for the desired layer
3. Use ZMK keycodes from `dt-bindings/zmk/keys.h`
4. Commit and push to trigger GitHub Actions build

## Troubleshooting

### Reset Settings
Build and flash the settings_reset shield to clear all settings:
```bash
west build -p -d build/reset -b piantor_pro_bt_left -- -DSHIELD=settings_reset
```

### Bluetooth Issues
- Use layer 3 (bt layer) to clear bonds: `&bt BT_CLR`
- Select different profiles: `&bt BT_SEL 0-4`
- Ensure both halves are paired to the same profile

### Build Failures
- Check `build.yaml` for correct board/shield combinations
- Verify West manifest in `config/west.yml` points to valid ZMK revision
- Review GitHub Actions logs for specific error messages